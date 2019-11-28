## 一 Docker compose简介

在实际开发中，业务可能需要被拆分成多个子任务，然后对这些子任务进行顺序组合，当子任务按照方案执行完毕后，就完成了业务目标。  

任务编排，就是对多个子任务执行顺序进行确定的过程。  

常见的任务编排工具：
- 单机版： docker compose
- 集群版：
  - Docker swarm Docker
  - Mesos Apache
  - Kubernetes（k8s） Google

Docker compose是一种docker容器的任务编排工具，官方地址：https://docs.docker.com/compose/   

## 二 compose 快速入门

docker compose的配置文件是：`docker-compose.yml`。  

docker compose 安装:
```
#安装依赖工具
sudo apt-get install python-pip -y
#安装编排工具
sudo pip install docker-compose
#查看编排工具版本
sudo docker-compose version
#查看命令帮助
docker-compose --help
```

PIP 源问题:
```
#用pip安装依赖包时默认访问https://pypi.python.org/simple/，
#但是经常出现不稳定以及访问速度非常慢的情况，国内厂商提供的pipy镜像目前可用的有：
#在当前用户目录下创建.pip文件夹
mkdir ~/.pip
#然后在该目录下创建pip.conf文件填写：
[global]
trusted-host=mirrors.aliyun.com
index-url=http://mirrors.aliyun.com/pypi/simple/
```

compose简单配置文件:
```
#创建compose文件夹
mkdir -p ./docker/compose
#进入到文件夹
cd ./docker/compose
#创建yml文件
vim docker-compose.yml

# 内容
version: '2'
services:
    web1:
        image: nginx
        ports:
            - "9999:80"
        container_name: nginx-web1
    web2:
        image: nginx
        ports:
            - "8888:80"
        container_name: nginx-web2
```

运行一个容器：
```
#后台启动：
docker-compose up -d
#注意：
#如果不加-d，那么界面就会卡在前台
#查看运行效果
docker-compose ps
```

compose服务命令:
```
#后台启动：
docker-compose up -d
#删除服务
docker-compose down
#查看正在运行的服务
docker-compose ps

#启动一个服务
docker-compose start <服务名>
#注意：
#如果后面不加服务名，会停止所有的服务
#停止一个服务
docker-compose stop <服务名>
#注意：
#如果后面不加服务名，会停止所有的服务
#删除服务 这个docker-compose rm不会删除应用的网络和数据卷。工作中尽量不要用rm进行删除
docker-compose rm

#查看运行的服务
docker-compose ps
#查看服务运行的日志
docker-compose logs -f
#注意：
#加上-f 选项，可以持续跟踪服务产生的日志
#查看服务依赖的镜像
docke-compose images
#进入服务容器
docker-compose exec <服务名> <执行命令>
#查看服务网络
docker network ls
```