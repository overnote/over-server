## 一 Nginx介绍

Nginx现在的功能十分强大，可以作为HTTP服务器，反向代理服务器，缓存加速访问，邮件服务器，支持FastCGI、SSL、Virtual Host、URL Rewrite、Gzip等常见功能，并支持第三方模块扩展。  

Nginx在生产环境中，能够支持高达4万的并发连接数。这得与Nginx使用最新的epoll(Linux2.6内核)和kqueue(freebsd)网络I/O模型。而Apache使用的是传统的select模型，其Prefork模式为多进程模式，需要经常派生子进程，消耗CPU比Nginx高很多。  

## 二 Nginx安装

#### 2.1 win安装

nginx的win安装包是绿色的，解压就可以直接使用：
```
start .\nginx.exe               # 启动后查看进程管理器中是否有nginx进程，或者访问127.0.0.1
```
但是有可能遇到无法启动问题，大多是因为80端口被占用，解决办法：  
打开regedit，找到：HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\HTTP，找到REG_DWORD类型的项Start，将其改为0，重启电脑即可。
常用命令：
```
nginx -s stop                   //停止nginx
nginx -s reload                //重新加载nginx
nginx -s quit                  //退出nginx
```
#### 2.2 Linux源码安装

CentOS安装：
```
# 安装依赖
yum install pcre pcre-devel zlib zlib-devel     # pcre是正则库，nginx重写url需要
yum install openssl openssl-devel               # 如果使用了https，需要该库   

# 下载
wget https://nginx.org/download/nginx-1.14.2.tar.gz

# 安装
tar zxvf nginx-1.14.2.tar.gz
cd nginx-1.14.2
./configure                  # 此处可以选择添加安装可选项，默认 --prefix=/usr/local/nginx --sbin-path=<prefix>/sbin/nginx              make && make install  
                                                    
```
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

#### 2.3 配置邮件代理安装

推荐的邮件代理配置为：
```
./configure --with-mail --with-mail_ssl_module --with-openssl=${BUILD_DIR}/openssl-1.0.1p
```
常见邮件代理安装配置：  

| 配置可选项 | 可选项说明 |
| ------ | ------ |
| --with-mail | 启用mail模块，该模块默认没有被激活 |
| --with-mail_ssl_module | 启用ssl |
| --without-mail_pop3_module | 启用mail模块后，单独禁用POP3模块 | 
| --without-mail_imap_module | 启用mail模块后，单独禁用IMAP模块 |
| --without-mail_smtp_module | 启用mail模块后，单独禁用SMTP模块 |
| --without-http | 禁用http模块，只使用mail | 

## 三 Nginx文件目录

在Nginx安装目录中，存在四个目录：
```
conf    配置文件目录
html    静态文件目录
logs    日志文件目录
sbin    命令目录
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

## 五 信号量

#### 5.1 常用信号量

- TERM：快速关闭
- INT：快速关闭
- QUIT：优雅的关闭进程（等请求结束后再关闭）
- HUP：重要！！！改变配置文件时用来，平滑的重读配置文件
- USR1：重读日志，在日志按月/日分割时使用
- USR2：平滑升级Nginx版本
- WINGCH：优雅关闭旧进程，配合USR2进行升级

#### 5.2 平滑重启

在对nginx进行平滑重启前，可以先确认其配置文件nginx.conf的语法是否正确：
```
nginx/sbin/nginx -t -c nginx.conf           # -t 就是用来检查语法是否正确的
```

执行平滑重启：
```
kill -HUP pid   # pid其实也被nginx记录了下来，可以使用该命令：kill -HUP `cat logs/nginx.pid`
```

#### 5.3 平滑升级

- 步骤1：替换nginx文件为新版文件  
- 步骤2：使用指令 `kill -USER2 pid`   
- 步骤3：旧版的主进程将重命名.pid文件为.oldbin，然后执行新版的nginx可执行程序，依次启动新的主进程和新的工作进程。此时新旧实例会同时运行。  
- 步骤4：从容关闭旧版 `kill -WINCH 旧版pid`，此时旧版工作进程会在处理所有连接后退出。  


