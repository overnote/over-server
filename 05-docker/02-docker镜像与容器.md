## 一 docker架构简介

Docker采用C/S架构，其引擎是模块化的，由许多工具协同工作，其主要组件包括：
- Docker Client：Docker客户端工具
- Docker daemon：Docker守护进程，实现了Docker的API。在Linux上，客户端与daemon之间通过IPC/UNIX Socket完成（/var/run/docker.sock）。
- containerd：用于操作新容器，如：容器的生命周期管理、镜像管理，创建容器的过程其实是fork一个runc实例。可以被第三方工具直接使用，如K8s中默认的容器运行时即是containerd。
- runc：OCI容器运行时规范的一个实现，是一个完全独立的容器运行时工具
- shim：容器解耦工具。在旧时代docker模型中，所有运行时逻辑在daemon中实现，启动和停止daemon会导致宿主机所有运行中的容器被kill！！！shim实现了无daemon容器，用于将运行中的容器与daemon解耦，方便daemon升级。

如图所示：  
![](../images/docker/docker-01.png)  

学习Docker需要了解的四大核心技术:
- 镜像（image）
- 容器（container）
- 数据卷（data volumes） 
- 网络（network）

## 二 docker镜像

### 2.1 镜像命令

镜像是docker的可执行文件，其中包括运行应用程序所需的所有代码内容、依赖库、环境变量和配置文件等，通过镜像才能创建容器。  

镜像常用命令有（镜像名可以是 镜像ID:镜像版本）：
```
docker images                  # 列出本地镜像，添加 -a 会列出已删除镜像，添加 镜像名 则只列出该名称的镜像 
docker search 镜像名            # 搜索镜像    
docker pull 镜像名              # 下载镜像，默认存储于/var/lib/docker
docker inspect 镜像名           # 查看镜像详细信息
docker tag 原镜像名 新镜像名      # 修改镜像名，示例：docker tag nginx:latest my-nginx:v1.0
docker rmi 镜像名               # 删除镜像，删除所有镜像示例：docker rmi `docker images -q`
docker save 导出镜像名 打包镜像名  # 将本地的一个或多个镜像打包保存成本地tar文件
docker load 本地镜像名           # 导入打包的一个镜像到本地
```

贴士：
- 如果一个image_id存在多个名称，那么应该使用 名称:版本 的格式删除镜像 
- 命令参数(OPTIONS)
  -  `-f`（--force）强制删除
  -  `-o`, --output string 指定写入的文件名和路径 

根据模板创建镜像：
```
# 登录系统模板镜像网站:https://download.openvz.org/template/precreated/
# 找到一个镜像模板进行下载，如: #https://download.openvz.org/template/precreated/ubuntu-16.04-x86_64.tar.gz 

# 命令格式:
cat 模板文件名.tar | docker import - [自定义镜像名]

# 演示效果:
cat ubuntu-16.04-x86_64.tar.gz | docker import - ubuntu-mini
```

### 2.2 镜像原理

Linux的文件系统由bootfs和rootfs两部分组成：
- bootfs：包含bootloader（引导程序）、kernel（内核）
- rootfs：即root文件系统，包含典型Linux系统中的`/dev`，`/proc`，`/bin`，`/etc`等目录和文件

不同的Linux发行版，其bootfs基本一样，而rootfs不同。  

Docker的镜像本质上市一个分层文件系统，如下所示：  

![](../images/cloud/docker-01.png)  

Docker镜像的最底端复用了宿主机的bootfs，第二层是root文件系统，称为base image，再往上逐级添加其他镜像文件。这种统一的文件系统（Union File System）技术能够将不同的层整合为一个文件系统，为这些层提供统一的视角，从用户角度来说，他们只会看到一个文件系统。  

一个tomcat镜像，需要从底往上依赖图中所示的多个其他镜像，如JDK镜像，所以tomcat的安装包才几十M，但是Docker中的tomcat安装后镜像却几百兆！  

同样的道理，Docker的centos镜像很小，远远不足1G，因为该镜像可以直接复用宿主机的bootfs，只单独使用了rootfs和其他镜像层。  

## 三 docker容器

### 3.0 docker容器概述

Docker将镜像文件运行起来后，产生的对象就是容器，容器相当于是镜像运行起来的一个实例，类似于常见的虚拟机的概念。**容器的本质其实是镜像在启动时，Docker在镜像最顶层加载了一个读写文件系统作为了容器！**  

容器与虚拟机的相同点：
- 容器和虚拟机一样，都会对物理硬件资源进行共享使用。 
- 容器和虚拟机的生命周期比较相似(创建、运行、暂停、关闭等等)。
- 容器中或虚拟机中都可以安装各种应用，如redis、mysql、nginx等。也就是说，在容器中的操作，如同在一个虚 拟机(操作系统)中操作一样。
- 同虚拟机一样，容器创建后，会存储在宿主机上:linux上位于/var/lib/docker/containers下

容器与虚拟机的不同点：
- 虚拟机的创建、启动和关闭都是基于一个完整的操作系统。一个虚拟机就是一个完整的操作系统。而容器直接运行在宿主机的内核上，其本质上以一系列进程的结合。
- 容器是轻量级的，虚拟机是重量级的。
  - 首先容器不需要额外的资源来管理，虚拟机额外更多的性能消耗;
  - 其次创建、启动或关闭容器，如同创建、启动或者关闭进程那么轻松，而创建、启动、关闭一个操作系统就没那么方便了。
- 也因为轻量，在给定的硬件上能运行更多数量的容器，甚至可以直接把Docker运行在虚拟机上。

### 3.1 容器常用命令

贴士：容器名和容器id的作用相同
```
docker ps                       # 查看容器列表，-a参数会显示所有运行过的镜像

docker run                      # 快速创建并启动一个新容器，如下所示：以centos镜像为基础，创建一个名为mycentos1的镜像
                                # docker run -i -t --name=mycentos1  centos:latest /bin/bash
                                # 参数 -i 表示将当前shell的 STDIN连接到容器上
                                # 参数 -t 表示分配虚拟终端tty，多个参数可以合并，如：-it
                                # 参数 -d 表示守护进程启动
                                # 参数 -p 映射容器的端口为宿主机的端口，如安装mysql可以 -p 3306:3306  前者为宿主机端口，后者为容器端口

exit                            # 退出容器：命令可以在容器内退出容器，回到宿主机，但是会造成容器运行停止

docker exec 容器名               # 使用该命令可以进入以守护进程创建的容器
                                # 示例：docker exec mycentos1 /bin/bash

docker create 容器名             # 创建容器，是docker run的分解动作之一
docjer start 容器名              # 启动容器，是docker run的分解动作之一，后面可以跟 容器名称或者容器ID    

docker inspect 容器名            # 查看容器详细信息
docker logs 容器名               # 查看容器日志
docker rename 容器名 新容器名称    # 重命名容器
docker port 容器名               # 查看容器端口

# 查看容器网络信息：
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 930f29ccdf8a
```

### 3.2  容器的关闭、终止、删除

```
docker stop 容器名              # 关闭：延迟关闭一个或多个处于暂停状态或者运行状态的容器 
docker kill 容器名              # 终止：强制并立即关闭一个或多个处于暂停状态或者运行状态的容器

docker rm 容器名                # 删除已关闭容器，-f 参数可以删除正在运行的容器
                               # 批量删除示例：docker rm -f $(docker ps -a -q)
```