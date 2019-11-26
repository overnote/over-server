## 一 日志配置

#### 1.1 日志配置的2个指令

访问日志位于主配置文件的http中，主要指令有两个：
- log_format：指定日志记录格式
- access_log：指定志存放路径，格式，缓存大小

设置如下：
```
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;
```

#### 1.2 log_format

log_format的`main`是日志记录形式的自定义名称，常见的配置：
| 配置项 | 配置说明 |
| --- | :--- |
| $remote_addr | 访问网站的客户端地址 | 
| $http_x_forwarded_for  | 当前端又代理服务器时，设置web节点记录客户端地址，此参数生效的前提是代理服务器上也要进行x_forwarded_for设置  |
| $remote_user | 远程客户端用户名 |
| $time_local | 访问时间与时区 |
| $request  | 请求方式  |
| $body_bytes_sents  | 响应body字节数  |
| $http_referer  |  从哪个链接过来，可以根据referer进行防盗链设置 |
| $status  | http状态码  |
| $http_user_agent  |  客户端访问信息,可以依此给用户渲染手机页面，电脑页面 |
  
注意：如果nginx作为了反向代理服务器，就无法获取客户端的真实IP了，因为`$remote_addr`变量拿到的是反向代理服务器的IP地址，但是反向代理服务器在转发请求的HTTP头信息中，可以增加`X-Forwarded-For`信息，用以记录原有的客户端IP地址和原来的客户端请求的服务器地址。  

#### 1.3 access_log

nginx允许针对不同的server做不同的日志处理：
```
server {
    access_log logs/test.log main       # 在指定的server下添加，需要打开http配置下的main名称
}
```

如果不想记录日志，可以使用 `access_log off;`关闭日志。

#### 1.4 错误日志

错误日志可以直接放置在http中，也可以放在server，location中：
```
server {
    listen   80;
    ....
    error_log   logs/error.log  error;      # error是日志的级别
}
```

#### 1.5 日志切割

nginx没有很好的日志切割工具，在日志目录下书写一个shell脚本：
```
vim runlog.sh

# 内容如下，运行方式 ssh runlog.sh
#!/bin/bash

LOGPATH=/usr/local/nginx/logs/test.log

## 一个月一个目录
BASEPATH=$BASEPATH/$(date -d yesterday +%Y%m)

mkdir -p $BASEPATH

bak=$BASEPATH/$(date -d yesterdata +%Y%m$d%H%M).test.log

mv $LOGPATH $bak

touch $LOGPATH

kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`
```

让该shell每分钟执行一次（生产环境每天执行一次或者按需求执行），命令行输入以下命令：
```
crontab -e 

# 添加以下内容
*/1 * * * * sh /usr/local/nginx/runlog.sh
```