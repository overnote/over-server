## 一 kubeadm安装方式部署k8s

### 1.1 安装方式概述

kubeadm是k8s集群的简易部署方式，避免源码安装中的大量复杂环节，且能自动生成证书。  

kubeadm集群版安装方式主要涉及的软件：
- kubelet：运行在所有节点上，负责启动Pod、容器
- kubeadm：运行在所有节点上，用于初始化节点
- kubectl：运行在Mster节点上，是k8s的命令行工具，用于部署管理应用，查看各种资源，创建、删除、更新各种组件

使用kubeadm方式部署的k8s集群与二进制方式部署最大的区别是：kubeadm部署的k8s集群中，除了kubelet以外所有组件都是以容器方式运行的，而二进制方式部署的k8s集群中的很多组件是以系统原生进程方式运行。  

### 1.2 安装环境概述

本案例部署的是单Master节点集群，只是用来学习，在生产环境中，必须支持Master节点的高可用。不过kubeadm工具可以方便的支持k8s集群从单Master节点扩展为多Master节点。  

案例中使用了VMware安装了三台虚拟主机(最小化安装，并勾选右侧的几个开发工具)：
- master1：centOS7系统，是管理节点，ip为 192.168.216.100
- worker1：centOS7系统，是工作节点，ip为 192.168.216.101
- worker2：centOS7系统，是工作节点，ip为 192.168.216.102

## 二 k8s部署前提

### 2.0 为集群所有机器分配IP

**如果使用真实的云服务器环境则可以忽略该步骤，不过记得要开放网络安全端口：4001,10250,2378,6443**。  

如果使用VMware安装了三台虚拟主机，此时需要配置虚拟主机的ip：
```
vim /etc/sysconfig/network-scripts/ifcfg-ens33

DEVICE="eth0"
TYPE="Ethernet"
ONBOOT="yes  "
BOOTPROTO="static"                # 此处也可以是none
IPADDR="192.168.216.100"          # 注意三台机器在本处设置的ip分别为100，101，102
PREFIX="24"
NETMASK="255.255.255.0"
GATEWAY="192.168.216.2"
DNS1="119.29.29.29"

# 重启网卡
systemctl restart network
```

### 2.1 映射集群ip与hostname

修改hostname：
```
# 分别修改三台主机的hostname
hostnamectl set-hostname master1    # 注意三台机器在本处设置的hostname分别为master1，worker1，worker2

# 查看修改结果
hostname
```

修改hosts：
```
# 集群每台机器都进行如下一致的修改
vim /etc/hosts

192.168.216.100   master1
192.168.216.101   worker1
192.168.216.102   worker2

# 测试
ping master1
```

贴士：如果使用的是真实云服务器环境，这里的`/etc/hosts`应该书写内网地址！  

### 2.2 关闭防火墙与selinux

所有节点必须关闭防火墙、selinux：
```
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld 

# 关闭selinux
sed -i 's/enforcing/disabled/' /etc/selinux/config
setenforce 0
getenforce                  # 重启后查看selinux关闭情况
```

### 2.3 集群所有机器的时间同步

集群机器都需要进行时间同步，这里使用ntpdate：
```
# 安装配置ntpdate
yum install -y ntpdate
crontab -e                              
0 */1 * * * ntpdate time1.aliyun.com  # 编辑同步配置，这里设置1小时同步一次

# 直接同步一次
ntpdate time1.aliyun.com
```

### 2.4 集群所有机器的swap分区关闭

kubeadm部署需要关闭swap分区（k8s认为swap影响性能）：
```
echo "vm.swappiness = 0">> /etc/sysctl.conf 
swapoff -a && swapon -a
sysctl -p
```

### 2.5 桥接的IPV4流量传递到iptables的链

RHEL / CentOS 7上的一些用户报告了由于iptables被绕过而导致流量路由不正确的问题，书写网桥过滤配置文件：
```
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system
```

### 2.6 集群所有机器部署ipvs

ipvs比iptables效率更高，所以这里使用ipvs：
```
# 安装ipvs
yum install -y ipset ipvsadm

# 添加需要加载的模块
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF

# 修改文件权限并执行
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules

# 检查是否加载
lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

### 2.7 重启

由于修改swap分区等一些设置需要重启，所以笔者这里将重启放在最后一步：
```
reboot    
```

## 三 安装必要软件

### 3.1 所有节点安装并启动docker

集群所有机器都需要安装docker-ce，但是必须要注意的是docker的版本与k8s的版本具有对应关系，本章节安装的docker18与k8s1.15对应：
```
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
yum -y install docker-ce-18.06.1.ce-3.el7
systemctl start docker
docker --version
```

配置docker镜像的地址以及cgroup：
```
vim /etc/docker/daemon.json

{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "registry-mirrors": ["https://7j9793w7.mirror.aliyuncs.com"]
}
```

贴士：在较低版本的docker中，如果要进行docker启动定制化，还需要将文件`/usr/lib/systemd/system/docker.service`中`ExecStart=/usr/bin/dockerd`的`-H`及其之后的内容注释掉。   

重启docker，并设置为开机启动：
```
systemctl daemon-reload
systemctl restart docker
systemctl enable docker 
```

### 3.2 所有节点安装k8s工具

先设置k8s本身的yum下载源：
```
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

安装k8s工具，注意这里一定要看一下版本号，因为 kubeadm init 的时候 填写的版本号不能低于k8s版本：
```
# 安装指定版本，worker节点无需安装kubectl
yum install -y kubelet-1.15.0 kubeadm-1.15.0 kubectl-1.15.0
```

贴士：Ubuntu在下载k8s时，会报一个 NO_PUBKEY 错误，解决方案：`gpg --keyserver keyserver.ubuntu.com --recv-keys ***   # 这里的 *** 是key的后8位`

### 3.3 所有节点启动kubelet服务

启动k8s服务kubelet：
```
# 修改kubelet配置
vim /etc/sysconfig/kubelet

KUBELET_EXTRA_ARGS=--cgroup-driver=systemd
KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/ --cni-bin-dir=/opt/cni/bin

# 启动kubelet服务
systemctl enable  kubelet 
systemctl start  kubelet  
systemctl status kubelet.service                       # 查看服务状态
```

发现服务并未正常启动，具体原因可以使用命令：`journalctl -xefu kubelet` 查看。这里要注意的是：
- 如果启动失败是因为“/var/lib/kubelet/config.yaml”文件不存在，则不用处理，k8s集群在接下来的步骤中初始化时会自动创建此文件
- 如果是说 master not found等错误，则需要前往配置文件`/etc/kubernetes/`修改所有的ip地址为当前主机ip

## 四 k8s集群启动

### 4.1 Master节点初始化

Master节点初始化，即执行`kubeadm init`命令，该命令会自动拉取组件、启动kubelet、生成证书（包括etcd、apiserver、proxy）、生成kubeconfig文件、将这些组件作为静态Pod启动。   

操作具体命令（这里的kubernetes-version 一定要和上面安装的版本号一致 否则会报错）：
```
kubeadm init \
--apiserver-advertise-address 192.168.216.100 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.15.0 \
--service-cidr=10.1.0.0/16 \
--pod-network-cidr=10.244.0.0/16
```

**注意**：
- 这里的apiserver地址如果部署在类似阿里云上，应该使用内网地址！
- 初始化失败则可以重置后重试 `kubeadm reset`
- `kubeadm config image list`可以查看当前要下载的镜像情况

### 4.1.1 Master节点初始化-从配置文件初始化

4.1的初始化命令过于冗长，可以创建一个配置文件，从自定义的配置文件进行初始化：
```
# 创建配置文件
mkdir /soft/k8s -p
cd /soft/k8s
kubeadm config print init-defaults ClusterConfiguration > /soft/k8s/kubeadm.conf

# 修改配置文件中的一些内容
vim /soft/k8s/kubeadm.conf

localAPIEndpoint:
  advertiseAddress: 192.168.216.100
  bindPort: 6443
imageRepository: registry.aliyuncs.com/google_containers
kubernetesVersion: v1.15.0
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12

# 从该配置文件下载相应软件，包括：kube-apiserver、kube-controller-manager、kube-scheduler、kube-proxy、etcd等
kubeadm config images pull --config /soft/k8s/kubeadm.conf

# 初始化
kubeadm init --config /soft/k8s/kubeadm.conf
```

### 4.2 记录token

初始化成功后，会出现token提示，请记住显示的token，后面再节点加入集群时会用到：
```
kubeadm join 192.168.216.100:6443 --token k8mswg.ln0wjibvne8exzh5 \
    --discovery-token-ca-cert-hash sha256:611d1c5a3d77e4d6d25bae0dd50a76735318357a26adc8f21154b6a34410aa1d 
```

### 4.3 Master修正apiseverlocalhost配置

集群初始化完毕后，使用查看节点的命令：`kubectl get nodes`，仍然会有异常报出，这是因为kube-apiserver默认使用的是localhost，这也会造成安装网络插件的错误，需要进行如下操作：
```
# 如果是普通用户需要进行如下操作
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
systemctl restart kubelet

# 如果是root用户，也可以这样做：
export KUBECONFIG=/etc/kubernetes/admin.conf
systemctl restart kubelet
```

设置完毕后，重新查看K8S的状态：
```
kubectl get nodes   

# 显示结果为：
NAME      STATUS     ROLES    AGE   VERSION
master1   NotReady   master   60m   v1.15.0
```

### 4.4 Master节点安装网络插件 

未准备好的k8s集群需要网络支持，即额外安装Pod网络插件 CNI：
```
# 安装网络插件
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml

# 反复查看下载速度，如果下载速度很慢，可将该文件下载下来，修改文件内的镜像地址，重新 kubectl apply -f 本地yml位置
kubectl get pods -n kube-system 

# 下载完毕后（需要一定时间！），再次查看状态（
kubectl get nodes

NAME      STATUS   ROLES    AGE   VERSION
master1   Ready    master   71m   v1.15.0
```

## 五 Worker节点 执行加入 k8s集群

### 5.1 加入步骤

去worker节点执行向集群添加新节点（注意：node节点需要安装kubeadm）：
```
kubeadm join 192.168.216.101:6443 --token k8mswg.ln0wjibvne8exzh5 \
    --discovery-token-ca-cert-hash sha256:611d1c5a3d77e4d6d25bae0dd50a76735318357a26adc8f21154b6a34410aa1d 
```

加入成功后，会提示回到master节点，查看节点状态：
```
kubectl get nodes 

NAME      STATUS   ROLES    AGE    VERSION
master1   Ready    master   113m   v1.15.0
worker1   Ready    <none>   68s    v1.15.0
```

此时worker1如果一直notready，这是因为每个节点都需要启动若干组件，这些组件都是在 Pod 中运行，则可以在master上执行
```
# 查看worker的镜像的下载情况
kubectl get pod --all-namespaces
```

### 5.2 无法加入的一些问题解决

可以通过在master删除worker节点后重新加入：
```
# master执行删除
kubectl delete node worker1

# worker执行重置
kubeadm reset

# worker执行再次加入口令
```

如果无法加入，可能是秘钥过期，可以在master重新生成：
```
# 在master重新生成一个24小时有效期的口令
kubeadm token create

# 在worker中重新执行上述加入口令
```

如果创建的节点已经加入，因为一些故障，宕机后要重新加入，则需要删除上次加入时新生成的三个文件，然后重新加入：
```
rm -rf /etc/kubernets/bootstrap-kubelet.conf
rm -rf /etc/kubernets/kubelet.conf
rm -rf /etc/kubernets/pki/ca.crt
```