## 一 常见应用在docker中的部署

### 1.0 应用安装贴士

容器内的网络服务是不能与外部直接进行通信的，需要进行端口映射

### 1.1 部署mysql

```
# 拉取mysql5.7镜像
docker pull mysql:5.7

# 启动容器，并将其端口映射在宿主机的3307端口上
docker run -id --name=mymysql \
-p 3307:3306 \
-e MYSQL_ROOT_PASSWORD=123456 \
mysql:5.7

# 当然还可以添加一些命令，如：
-v /soft/mysql/conf:/etc/mysql/conf.d \
-v /soft/mysql/logs:/logs \
-v /soft/mysql/data:/var/lib/mysql \
```

### 1.2 部署nginx

```
docker pull nginx

docker run -id --name=mynginx \
-p 80:80 \
-v /soft/nginx/conf/nginx.conf:/etc/nginx/nginx.conf
-v /soft/nginx/log:/var/log/nginx \
-v /soft/nginx/html:/usr/share/nginx/html \
nginx
```

### 1.3 部署tomcat

```
docker pull tomcat 

docker run -id --name=mytomcat \
-p 8080:8080 \
-v /usr/local/tomcat/webapps \
tomcat
```

### 1.4 部署redis

```
docker pull redis:5.0

docker run -id --name=myredis -p 6379:6379 redis:5.0
```

## 二 镜像的制作

### 2.0 镜像的制作方式

镜像有两种制作方式：
- 基于容器直接制作镜像
- 使用dckerfile制作镜像

本章节只简要介绍使用容器制作镜像的方式，这里即可以直接升级以前的镜像，也可以将当前容器导出为一个镜像。

### 2.1 容器版本升级为镜像

方式一：将容器版本升级，命令为：`docker commit -m '改动信息说明' -a "作者信息" [container_id][new_image:tag] `

```
#  假设将名为 mytomcat 的容器（容器id为 340ea66）制作一个镜像  img_mytomcat
docker commit 340ea66 img_mytomcat:1.0          # 不写版本时默认为latest
docker images                                   # 此时可以查看docker的镜像是否存在img_mytomcat
```

注意：此方式适用于镜像内部做了一定修改后，再次生成为新的镜像！  

产生的新镜像也可以进行压缩，用于在不同人员之间传递，docker也提供了镜像文件的压缩功能：
```
# 压缩镜像
docker save -o img_mytomcat.tar img_mytomcat:1.0

# 解压镜像：将压缩文件解压为一个新的镜像
docker load -i img_mytomcat.tar
```

当然上述的save、load的命令也常用于docker的备份、迁移！  

### 2.2 容器导出镜像

命令格式: `docker export [容器id] > 模板文件名.tar`，演示如下：

```
docker export ae63ab299a84 > nginx.tar        # 导出镜像为一个模板文件 nginx.tar
cat nginx.tar | docker import - panda-test    # 导入模板 nginx.tar，并将其镜像名称重新设定为 panda-test
```

export和save命令都产生了一个tar压缩文件，但是二者是有区别的，export导出的镜像要比save保存的镜像小，因为export导出的镜像在导入时会丢失镜像的历史。  

import和load命令都可以从一个tar压缩文件产生新的镜像，但是import可以额外指定镜像的名字。