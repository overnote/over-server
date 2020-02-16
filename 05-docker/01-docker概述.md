## 一 Docker简介

容器与虚拟机相比，主要区别在于容器是应用层级的隔离，虚拟化是物理资源层面的隔离：
```
容器运行不会独占操作系统，运行在同一个宿主机上的多个容器其实是共享着宿主机系统
```

Docker是一款使用Go语言开发的基于LCX容器技术开源容器引擎，docker主要解决的问题：
- 保证程序运行环境的一致性; 
- 降低配置开发环境、生产环境的复杂度和成本; 
- 实现程序的快速部署和分发。

开发人员常说的Docker是指Docker核心引擎，Docker引擎是用户来运行和管理容器的核心软件，基于该引擎产生的组件，以及周边大量的第三方服务共同构成了当前Docker的生态。  

## 二 安装docker

### 2.0 安装贴士

Docker有两个版本：企业版EE和社区版CE（免费）。   

安装docker的前提条件：
- 只能是x86_64和amd64架构计算机，docker不支持32位CPU
- Linux要求：
  - Linux内核需要在3.8及以上，查看版本：`uname -a`
  - 内核必须支持一种适合的存储驱动（storage driver），如：DeviceManger，vfs等，查看命令：`ls -l /sys/class/misc/device-mapper`
  - 内核必须支持并开启cgroup和namespace
- Win要求：
  - 必须是win10企业版或者教育版
  - 需要开启Hyper-V等容器特性：右键单击开始-应用和功能-程序和功能-启用或关闭Windows功能-勾选Hyper-V和容器，然后重启
- Mac说明：mac版docker是基于LinuxVM制作的，但是足够满足开发与测试使用

贴士：docker安装完毕后会多出docker0网卡。

### 2.1 Win/Mac安装docker

直接安装，下一步下一步即可，测试安装结果：
```
docker --version
```

安装完毕后的docker还会提供一个镜像的图形化管理软件下载地址，名为：Kitmatic，该软件也推荐安装。  

### 2.2  CentOS安装

官方安装文档：https://docs.docker.com/install/linux/docker-ce/centos/  

安装步骤：
```
# 移除旧版本docker
sudo yum remove docker \
                docker-common \
                docker-selinux \
                docker-engine

# 创建docker用户组
sudo groupadd docker                # docker会自动查找该用户组
sudo gpasswd -a ${USER} docker      # 添加当前用户到用户组，可以避免命令中需要sudo的麻烦
newgrp docker                       # 更新用户组

# 安装必要组件
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# 配置软件源
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 检查 /etc/yum.repos.d/docker-ce.repo 中的url是否都是阿里云的，可把 download-stage.docker.com 替换为 mirros.aliyun.com/docker-ce   

# 更新并安装docker-ce
sudo yum makecache fast
sudo yum -y install docker-ce       # docker-ce 是社区版，免费

# 检测安装
docker version
```

## 三 docker启动镜像加速

docker的镜像加速其实就是修改`/etc/docker/daemon.js`文件内容，然后重启docker即可，常用的镜像官网有：
- https://www.daocloud.io/mirror 
- https://cr.console.aliyun.com/cn-beijing/instances/mirrors

## 四 docker常见使用

docker的启动相关命令：
```
# centOS6
sudo service docker start           # 直接启动
sudo service docker enable          # 开机启动

# centOS7
sudo systemctl start docker         # 守护进程启动
sudo systemctl enable docker        # 开机启动
sudo systemctl stop docker          # 守护进程停止
sudo systemctl restart docker       # 重启docker
sudo systemctl status docker        # 查看docker状态
```

docker的基本目录：
```
/etc/docker/        # docker的认证目录
/var/lib/docker/    # docker的应用目录
```

## 五 docker升级

```
# 列出包含docker字段的软件的信息
rpm -qa | grep docker

# 卸载
yum remove docker-1.13.1-53.git774336d.el7.centos.x86_64
yum remove docker-client-1.13.1-53.git774336d.el7.centos.x86_64
yum remove docker-common-1.13.1-53.git774336d.el7.centos.x86_64

# 升级到最新版
curl -fsSL https://get.docker.com/ | sh

# 重启并设置开机启动
systemctl restart docker
systemctl enable docker
```