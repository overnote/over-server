## 一 Nginx介绍

Nginx现在的功能十分强大，可以作为HTTP服务器，反向代理服务器，邮件服务器等，支持FastCGI、SSL、Virtual Host、URL Rewrite、Gzip等常见功能，并支持第三方模块扩展。  

Nginx在生产环境中，能够支持高达4万的并发连接数，这归功于与Nginx使用最新的epoll(Linux2.6内核)和kqueue(freebsd)网络I/O模型。而Apache使用的是传统的select模型，其Prefork模式为多进程模式，需要经常派生子进程，消耗CPU比Nginx高很多。  

Nginx常见的使用场景：
- 静态资源服务：通过本地文件系统提供服务
- 反向代理服务：缓存加速、负载均衡
- 传统API服务：利用OpenResty提供

Nginx的优点：
- 高并发、高性能
- 扩展性好，有相当多的第三方插件
- 高可靠性
- 热部署
- BSD开源许可，支持商业修改许可

## 二 Nginx安装

### 2.1 win安装

nginx的win安装包是绿色的，解压就可以直接使用：
```
start .\nginx.exe               # 启动后查看进程管理器中是否有nginx进程，或者访问127.0.0.1
```
但是有可能遇到无法启动问题，大多是因为80端口被占用，解决办法：  
- 打开regedit，找到：HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\HTTP
- 继续找到REG_DWORD类型的项Start，将其改为0，重启电脑即可

### 2.2 Linux源码安装

CentOS安装：
```
# 安装依赖
yum install pcre pcre-devel zlib zlib-devel     # pcre是正则库，nginx重写url需要
yum install openssl openssl-devel               # 如果使用了https，需要该库   

# 下载
wget https://nginx.org/download/nginx-1.14.2.tar.gz

# 解压缩
tar zxvf nginx-1.14.2.tar.gz
cd nginx-1.14.2

# 此处可以选择添加安装可选项，
# 默认 --prefix=/usr/local/nginx --sbin-path=<prefix>/sbin/nginx
# 开启该模块，则可以支持https：–with-http_ssl_module
# 要查看支持哪些模块，可以使用命令 ./configure --help | more
./configure 

# 安装
make && make install                                                      
```

### 2.3 安装时的一些可选项

常用的./configure时可选参数：  

| 配置可选项 | 可选项说明 |
| ------ | ------ |
| --prefix=`<path>` | nginx安装根路径，默认位于/usr/local |
| --sbin- path=`<path>` | 指定nginx二进制文件路径，没有指定则依赖于prefix |
| --conf- path=`<path>` | 指定配置文件位置 |
| --error-log -path=`<path>` | 指定nginx错误文件写入目录 |
| --pid- path=`<path>` | 指定的文件将会写入nginxmaster进程的pid，通常位于/var/run下 |
| --lock- path=`<path>` | 共享存储器互斥锁文件的路径 |
| --user=`<user>` | worker进程运行的用户 |
| --group=`<group>` | worker进程运行的组 |
| --with-file-aio | 为FreeBSD4.3+和Linux2.6.22+系统启动异步I/O |
| --with-debug | 启用调试日志，生产环境不推荐 |


Nginx模块位于源码的src目录下，每个模块都符合模块化的思想：单一职责原则。  

Nginx的模块经常被划分为：
- 核心模块：提供进程管理、权限控制、错误日志、配置解析、事件驱动机制、正则表达式解析等
- 标准HTTP模块：快速编译Nginx后包含的模块，提供基本HTTP服务，高级HTTP服务
- 可选HTTP模块：默认不被编译进nginx，需要使用--with，用于扩展HTTP功能，处理一些特殊请求
- 邮件服务模块：当前版本默认编译不会将邮件服务编译进nginx
- 第三方模块：扩展Nginx服务器应用 

### 2.4 Nginx源码文件目录

在Nginx安装目录中，存在四个目录：
- conf：配置文件目录，nginx.conf是核心配置文件
- html：静态文件目录
- logs：日志文件目录，access.log是访问日志，err.log是错误日志
- sbin：命令目录

## 三 Nginx常用命令
```
nginx -s stop                   # 停止nginx
nginx -s reload                 # 重新加载nginx
nginx -s quit                   # 退出nginx
```

## 四 Nginx启动与关闭

Nginx服务在运行时，会保持一个主进程和一个或多个工作进程，通过给Nginx服务的主进程发送信号就可以控制服务的启动停止了。  

启动：
```
cd /usr/local/nginx/sbin
./nginx               # 可以添加 -c /usr/local/nginx/conf/nginx.conf  参数指定启动配置，未指定则默认加载安装目录conf中的nginx.conf
```

查看Nginx运行情况：Nginx的主进程（master）不负责处理网络请求，负责生成和管理多个子进程，子进程（work）用于处理网络请求。
```
ps aux|grep nginx                               # 此时可以看到Nginx拥有主进程和工作进程
cat /usr/local/nginx/logs/nginx.pid             # 该方法也可以查看nginx主进程

# 如果出现80端口被占用的情况，解决办法
netstat -antp                                   # 查看系统端口占用情况
kill -9 (PID)
```

关闭Nginx：关闭nginx的方式有很多种，一般使用发送系统信号给Nginx主进程的方式来停止
```
kill -INT PID                                   # 此处PID为Nginx的master进程PID
kill -9 nginx                                   # 强制停止所有nginx进程
```