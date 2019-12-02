## 一 kubeadm安装方式部署k8s

这里使用kubeadm集群版安装方式，主要涉及的软件：
- kubelet：运行在所有Cluster节点上，负责启动Pod、容器
- kubeadm：用于初始化Cluster
- kubectl：k8s的命令行工具，用于部署管理应用，查看各种资源，创建、删除、更新各种组件

k8s的部署和大多分布式系统要求一致：集群的各个节点之间可以互相通信。 

### 2.1 所有机器的安装前提

所有节点都要进行一些前提设置：
```
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld 

# 关闭selinux
sed -i 's/enforcing/disabled/' /etc/selinux/config
setenforce 0

# 关闭swap，k8s认为swap影响性能
echo "vm.swappiness = 0">> /etc/sysctl.conf 
swapoff -a && swapon -a
sysctl -p

# 修改主机名
hostname node01                         # 每个主机依次修改为 node02，node03，临时修改                       
hostnamectl set-hostname node01         # 每个主机依次修改为 node02，node03，永久修改

# 添加ip
vim /etc/hosts

172.17.38.208   node01  node01
172.17.219.75   node02  node02
172.17.219.76   node03  node03

# 讲桥接的IPV4流量传递到iptables的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system
```

### 2.2 所有机器安装docker

每个节点都要安装docker：
```
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
yum -y install docker-ce-18.06.1.ce-3.el7
systemctl enable docker && systemctl start docker
docker --version
```

### 2.3 所有机器安装k8s工具

设置YUM软件源：
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

安装kubeadm，kubelet和kubectl：
```
yum install -y kubelet-1.15.0 kubeadm-1.15.0 kubectl-1.15.0
systemctl start kubelet                                # 启动服务
systemctl status kubelet.service                       # 查看服务状态：activiting

vim /etc/sysconfig/kubelet

KUBELET_EXTRA_ARGS=--fail-swap-on=false
KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/ --cni-bin-dir=/opt/cni/bin

systemctl daemon-reload
systemctl restart kubelet
systemctl enable kubelet

systemctl status kubelet.service                    # 再次查看重启后的状态
```

### 2.4 master节点 初始化 k8s

master上执行 `kubeadm init`：会自动拉取组件、启动kubelet、生成证书（包括etcd、apiserver、proxy）、生成kubeconfig文件、将这些组件作为静态Pod启动
```
# 更新时间校验
ntpdate time.windows.com

# 初始化操作：如果本步骤失败则可以重置后重试 kubeadm reset
kubeadm init \
--apiserver-advertise-address=172.17.38.208 \
--image-repository registry.aliyuncs.com/google_containers \
--kubernetes-version v1.15.0 \
--service-cidr=10.1.0.0/16 \
--pod-network-cidr=10.244.0.0/16


# 初始化完毕后，会生成token需要在草稿中先记录下来
kubeadm join 172.17.38.208:6443 --token drtnet.qgkw8erp6vj1ik49 \
    --discovery-token-ca-cert-hash sha256:fcdb20705e2018d3a0c61118164070dfd4f57dd5d050ffbd83d346a25a1bd41d 

# 同时也提示需要做一些操作
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


此时k8s的状态是未准备的，需要额外安装网络安装Pod网络插件 CNI
```
# 查看节点状态
kubectl get nodes                   # 此时是未准备的

# 安装网络插件
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml

# 查看下载速度，
kubectl get pods -n kube-system 

# 如果下载速度很慢，可将该文件下载下来，修改问价内的镜像地址，重新 kubectl apply -f 本地yml位置

# 下载完毕后，再次查看状态
kubectl get nodes
```

### 2.5 node节点 加入 k8s

在执行kube init时，会输出如何join的内容，复制改内容，去node节点执行向集群添加新节点：
```
kubeadm join 172.17.38.208:6443 --token drtnet.qgkw8erp6vj1ik49 \
    --discovery-token-ca-cert-hash sha256:fcdb20705e2018d3a0c61118164070dfd4f57dd5d050ffbd83d346a25a1bd41d 
```

加入成功后，会提示你回到master节点，查看节点状态：
```
kubectl get nodes 
```



### 2.6 测试k8s集群

使用k8s部署一个nginx：
```
# 部署
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort

# 查看服务端口
kubectl get pod,svc
```

访问地址：http://node节点ip:80端口映射的端口号    

此时已经可以方便的做扩容了：
```
# 扩容nginx为3个副本
kubectl scale deployment nginx --replicas=3

# 查看扩容情况，并发性此时也上升了三倍
kubectl get pods
```


### 2.7 部署 Dashboard
```
# dashboard的yaml国内是无法访问的，可以下载后修改地址
wget https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

vim kubernetes-dashboard.yaml
# 修改镜像地址为： lizhenliang/kubernetes-dashboard-amd64:v1.10.1

# 默认Dashboard只能集群内部访问，修改Service为NodePort类型，暴露到外部：
kind: Service
apiVersion: v1
metadata:
labels:
  k8s-app: kubernetes-dashboard
name: kubernetes-dashboard
namespace: kube-system
spec:
type: NodePort
ports:
  - port: 443
    targetPort: 8443
    nodePort: 30001
selector:
  k8s-app: kubernetes-dashboard

# 修改完毕后也应用yaml文件
kubectl apply -f kubernetes-dashboard.yaml

# 查看dashboard运行情况：
kubectl get pods -n kube-system
```

访问地址：https://节点IP:30001   。  

访问后需要输入账户或者令牌，创建账户（名为 myadmin）并绑定默认cluster-admin管理员集群角色：
```
kubectl create serviceaccount myadmin -n kube-system
kubectl create clusterrolebinding myadmin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/myadmin/{print $1}')
```

使用输出的token登录Dashboard，不过该UI用户体验很差(LOL)，使用命令即可完成常用操作。  