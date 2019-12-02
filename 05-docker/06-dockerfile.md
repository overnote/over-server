## 一 Dockerfile简介

Dockerfile类似脚本，让容器实现自动化。  

Dockerfile的作用：
- 1、找一个镜像： ubuntu
- 2、创建一个容器： docker run ubuntu
- 3、进入容器： docker exec -it 容器 命令
- 4、操作： 各种应用配置....
- 5、构造新镜像： docker commit

Dockerfile使用规范：
- 1、大： 首字母必须大写D
- 2、空： 尽量将Dockerfile放在空目录中。
- 3、单： 每个容器尽量只有一个功能。
- 4、少： 执行的命令越少越好。

Dockerfile使用命令：
```
#构建镜像命令格式：
docker build -t [镜像名]:[版本号][Dockerfile所在目录]
#构建样例：
docker build -t nginx:v0.2 /opt/dockerfile/nginx/
#参数详解：
-t 指定构建后的镜像信息，
/opt/dockerfile/nginx/ 则代表Dockerfile存放位置，如果是当前目录，则用 .(点)表示
```

## 二 Dockerfile 快速入门

使用Dockerfile快速基于Centos创建一个定制化jdk1.8镜像：
```
# 创建Dockerfile专用目录
mkdir /home/docker/images/java -p
cd docker/images/java/
vim Dockerfile                          # 此名称固定
```

Dockerfile内容：
```
# 基础镜像
FROM centos:7
# 镜像作者
MAINTAINER ruyue ruyuejun@gmail.com
# 工作牡蛎
WORKDIR /usr
# 执行命令：将宿主机当前目录的jdk文件上传到容器目录
RUN mkdir /usr/local/java
ADD jdk-8u1710linux-x64.tar.gz /usr/local/java/
# 配置环境
ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
ENV JRE_HOME $JAVA_HOME/jre
```

构建：指定镜像名称与构建位置
```
docker build -t myjdk:1.8 .
```

## 三 常用命令

RUN：表示当前镜像构建时候运行的命令，如果有确认输入的话，一定要在命令中添加 -y，如果命令较长，那么可以在命令结尾使用 \ 来换行，生产中，推荐使用面数组的格式。
```
# shell模式，类似于 /bin/bash -c command
RUN <command>   
RUN echo hello

# exec 模式，类似于 RUN["/bin/bash", "-c", "command"]                  
RUN["executable", "param1", "param2"] 
RUN["echo", "hello"]
```

CMD：指定容器启动时默认执行的命令,每个Dockerfile只能有一条CMD命令，如果指定了多条，只有最后一条会被执行,如果在启动容器的时候使用docker run 指定的运行命令，那么会覆盖CMD命令。
```
#格式：
CMD ["executable","param1","param2"] (exec 模式)推荐
CMD command param1 param2 (shell模式)
CMD ["param1","param2"] 提供给ENTRYPOINT的默认参数；

# 示例
CMD ["/usr/sbin/nginx","-g","daemon off；"]
```

ENTRYPOINT：和CMD 类似都是配置容器启动后执行的命令，并且不会被docker run 提供的参数覆盖，每个Dockerfile 中只能有一个ENTRYPOINT，当指定多个时，只有最后一个起效。生产中我们可以同时使用ENTRYPOINT 和CMD，想要在docker run 时被覆盖，可以使用"docker run --entrypoint"
```
#格式：
ENTRYPOINT ["executable", "param1","param2"] (exec 模式)
ENTRYPOINT command param1 param2 (shell 模式)
```

ADD：将指定的`<src> `文件复制到容器文件系统中的`<dest>`，src 指的是宿主机，dest 指的是容器。所有拷贝到container 中的文件和文件夹权限为0755,uid 和gid 为0，如果文件是可识别的压缩格式，则docker 会帮忙解压缩。
```
# 格式：
ADD <src>... <dest>
ADD ["<src>",... "<dest>"]
```

注意：
- 1、如果源路径是个文件，且目标路径是以/ 结尾， 则docker 会把目标路径当作一个目录，会把源文件拷贝到该目录下;如果目标路径不存在，则会自动创建目标路径。
- 如果源路径是个文件，且目标路径是不是以/ 结尾，则docker 会把目标路径当作一个文件。如果目标路径不存在，会以目标路径为名创建一个文件，内容同源文件；如果目标文件是个存在的文件，会用源文件覆盖它，当然只是内容覆盖，文件名还是目标文件名。如果目标文件实际是个存在的目录，则会源文件拷贝到该目录下。注意，这种情况下，最好显示的以/ 结尾，以避免混淆。
- 如果源路径是个目录，且目标路径不存在，则docker 会自动以目标路径创建一个目录，把源路径目录下的文件拷贝进来。如果目标路径是个已经存在的目录，则docker 会把源路径目录下的文件拷贝到该目录下。
- 如果源文件是个压缩文件，则docker 会自动帮解压到指定的容器目录中

COPY:COPY 指令和ADD 指令功能和使用方式类似。只是COPY 指令不会做自动解压工作。单纯复制文件场景，Docker 推荐使用COPY。
```
#格式：
COPY <src>... <dest>
COPY ["<src>",... "<dest>"]
```

VOLUME：VOLUME 指令可以在镜像中创建挂载点，这样只要通过该镜像创建的容器都有了挂载点，通过VOLUME 指令创建的挂载点，无法指定主机上对应的目录，是自动生成的
```
#格式：
VOLUME ["/data"]
```

EVN：设置环境变量，可以在RUN 之前使用，然后RUN 命令时调用，容器启动时这些环境变量都会被指定
```
#格式：
ENV <key> <value> （一次设置一个环节变量）
ENV <key>=<value> ... （一次设置一个或多个环节变量）
```

WORKDIR：切换目录，为后续的RUN、CMD、ENTRYPOINT 指令配置工作目录。相当于cd.也可以使用多个WORKDIR 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。例如
```
#格式：
WORKDIR /path/to/workdir (shell 模式)
```

USER与ARG：指定运行容器时的用户名和UID，后续的RUN 指令也会使用这里指定的用户。如果不输入任何信息，表示默认使用root 用户
```
#格式：
USER daemon
ARG <name>[=<default value>]
```

注意：ARG 指定了一个变量在docker build 的时候使用，可以使用`--build-arg <varname>=<value>`来指定参数的值，不过如果构建的时候不指定就会报错。  

ONBUILD：触发器指令，当一个镜像A被作为其他镜像B的基础镜像时，这个触发器才会被执行，新镜像B在构建的时候，会插入触发器中的指令。使用场景对于版本控制和方便传输，适用于其他用户。
```
# 示例
ONBUILD COPY ["index.html","/var/www/html/"]
```

## 四 构架缓存

第一次构建很慢，之后的构建都会很快，因为他们用到了构建的缓存：
```
# 取消缓存：
docker build --no-cache -t [镜像名]:[镜像版本][Dockerfile位置]
```

## 五 实战-部署一个web服务

```
#1、docker环境配置
#1.1 获取docker镜像
#获取一个ubuntu的模板文件
cat ubuntu-16.04-x86_64.tar.gz | docker import - ubuntu-nimi
#1.2 启动docker容器
#启动容器，容器名称叫go-test
docker run -itd --name go-test ubuntu-nimi
#进入容器
docker exec -it go-test /bin/bash
#2、go环境部署
#2.1 基础环境配置
#配置国内源
vim /etc/apt/sources.list
#文件内容如下
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe
multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-security multiverse
#如果由于网络环境原因不能进行软件源更新可以使用如下内容
sudo sed -i 's/cn.archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
#更新软件源，安装基本软件
apt-get update
apt-get install gcc libc6-dev git vim lrzsz -y
#2.2 go环境配置
#安装go语言软件
//apt-get install golang -y
由于软件源问题改使用新版本go
将go1.10.linux-amd64.tar.gz拷贝到容器中进行解压
tar -C /usr/local -zxf go1.10.linux-amd64.tar.gz
#配置go基本环境变量
export GOROOT=/usr/local/go
export PATH=$PATH:/usr/local/go/bin
export GOPATH=/root/go
export PATH=$GOPATH/bin/:$PATH
#3、go项目部署
#3.1 获取beego代码
#下载项目beego
go get github.com/astaxie/beego
#3.2 项目文件配置
#创建项目目录
mkdir /root/go/src/myTest
cd /root/go/src/myTest
#编辑go项目测试文件test.go
package main
import (
"github.com/astaxie/beego"
) t
ype MainController struct {
beego.Controller
} f
unc (this *MainController) Get() {
this.Ctx.WriteString("hello world\n")
} f
unc main() {
beego.Router("/", &MainController{})
beego.Run()
} #
3.3 项目启动
#运行该文件
go run test.go
#可以看到：
#这个go项目运行起来后，开放的端口是8080
#4、测试
#4.1宿主机测试
#查看容器的ip地址
docker inspect go-test
#浏览器查看效果：
curl 172.17.0.2:8080
```