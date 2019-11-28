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

## 一 docker容器简介

Docker将镜像文件运行起来后，产生的对象就是容器，容器相当于是镜像运行起来的一个实例，类似于常见的虚拟机的概念。  

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

## 二 容器常用命令

### 2.1 容器常用命令

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

### 2.2  容器的关闭、终止、删除

```
docker stop 容器名              # 关闭：延迟关闭一个或多个处于暂停状态或者运行状态的容器 
docker kill 容器名              # 终止：强制并立即关闭一个或多个处于暂停状态或者运行状态的容器

docker rm 容器名                # 删除已关闭容器，-f 参数可以删除正在运行的容器
                               # 批量删除示例：docker rm -f $(docker ps -a -q)
```

### 2.3 文件拷贝 目录挂载

文件拷贝：
```
docker cp 宿主机内文件 容器名:容器目录  # 拷贝的目的是容器可能本身也需要安装一些软件，如容器mycentos内部安装redis
                                    # 该命令也可以用于从容器内拷贝出文件到本地，将cp后两个命令倒置即可
```

目录挂载：文件拷贝只能一个文件一个文件的传输，不很方便，可以将宿主机一个目录挂载为容器的目录：
```
docker run -id --name=mycentos2 -v /usr/local/mydir centos:latest   # 将宿主机的 mydir 挂载到容器 mycentos2中
```

## 三 几个软件安装示例

安装mysql：
```
docker pull mysql:5.7
docker run -di --name=mymysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
```

安装nginx
```
docker pull nginx
docker run -di --name=mynginx -p 80:80 nginx
docker exec -it mynginx /bin/bash
cd etc/nginx                                    # nginx在容器内的安装目录
```

## 四 基于容器创建镜像

方式一：将当容器版本升级
```
#命令格式: docker commit -m '改动信息' -a "作者信息" [container_id][new_image:tag] 

#命令演示:

# 第一步：执行一些操作
./docker_in.sh d74fff341687
mkdir /hello
mkdir /world
ls
exit

# 第二步：创建一个镜像
docker commit -m 'mkdir /hello /world ' -a "panda" d74fff341687 nginx:v0.2 

# 第三步：查看镜像，启动容器
docker images
docker run -itd nginx:v0.2 /bin/bash
./docker_in.sh ae63ab299a84           # 进入容器查看
ls
```

方式二：直接导出镜像
```
#命令格式: docker export [容器id] > 模板文件名.tar 

#命令演示:
docker export ae63ab299a84 > nginx.tar        # 导出
cat nginx.tar | docker import - panda-test    # 导入
```

import与load的区别: import可以重新指定镜像的名字，docker load不可以  

export 与 保存 save 的区别:
- export导出的镜像文件大小，小于save保存的镜像。
- export导出是根据容器拿到的镜像，再导入时会丢失镜像所有的历史。

