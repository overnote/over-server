## 一 服务编排

> 服务编排：在实际开发中，业务可能需要被拆分成多个子任务，然后对这些子任务进行顺序组合，当子任务按照方案执行完毕后，就完成了业务目标，服务编排即对多个子任务执行顺序进行确定的过程

常见的服务编排工具：
- docker swarm：docker公司出品的服务编排工具套件
- Mesos：
- kubernetes：即大名鼎鼎的k8s，其目标是让容器化应用的部署更加简单、高效！

## 二 Docker Compose安装

Docker Compose 是docker的一种服务编排工具，官方地址：https://docs.docker.com/compose/。  

目前Compose已经完全之多Linux、Mac、Win平台，不过在安装之前需要先确保已经安装了docker，安装步骤如下：
```
# 下载二进制包
curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose- `uname -s` - `uname -m` -o /usr/local/bin/docker-compose

# 设置文件可执行权限
chmod +x /usr/local/bin/docker-compose

# 查看安装
docker-compose -v
```

卸载操作：
```
rm /usr/local/bin/docker-compose
```

## 三 Docker Compose实战

需求：使用docker compose编排 nginx + goweb项目。  

docker compose的配置文件名必须是`docker-compose.yml`，书写该配置文件内容如下：
```yml
version: '3'
services:
  nginx:
    image: nginx
    ports:
      - 80:80
    links:
      - app
    volumes:
      - /soft/nginx/conf.d:/etc/nginx/conf.d
  app:
    image: hellogo
    expose:
      - "8080"
```

贴士，多个配置可以如下设置
```yml
    volumes:
      - /soft/nginx/conf.d:/etc/nginx/conf.d
      - /soft/nginx/log:/var/log/nginx
```

在配置文件所在目录中，使用docker-compose启动容器：
```
docker-compose up
```