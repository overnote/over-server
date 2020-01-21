## 一 核心配置nginx.conf简介

Nginx的核心配置文件位于`/conf/nginx.conf` ,常见设置有：

```
worker_processes  1;                        # 工作进程数，设置太多会争夺CPU，一般设为 CPU数*核心数

events {                                    # 一般配置Nginx连接特性
    use epoll                               # 使用哪种网络I/O模型，Linux推荐epoll，FreeBSD推荐kqueue
    worker_connections  1024;               # 1个work同时允许多少个连接
}

http {                                      # 配置虚拟主机，一般配置多个server

    include    mime.types;                  # 主配置文件nginx.conf包含了另外一个配置文件mime.types
    default_type   application/octet-stream;  
    sendfile   on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page 500 502 503 504 /50x.html
        location = /50x.html {
            root   html;
        }
    }

}
```

nginx核心配置的结构：
```
...
http {

    server {

    }

    server {
        
    }

    ...
}
```

该段配置的常见解读是：
- 每个server对应一个域名，有多少域名，就有多少个server，比如www.test.com,news.test.com,shop.test.com,此时nginx需要配置三个server。
- 一个server下有多个location，负责匹配url字符串

## 二 常见指令

- root:  表示资源根目录，可以使用Nginx大多变量（除了`$document_root`,`$realpath_root`）,该指令可以应用于http、server、location中。  
- index: 默认首页  
- error_page code 错误页面地址: 错误页面  
- allow: 语法结构为 `allow address | CIDR | all`，用于设置允许客户端访问的IP，不支持同时设置多个，如果要设置多个，需要重复使用allow指令，CIDR表示允许客户端访问的CIDR地址，如：202.80.18.23/25，all代表允许所有。
- deny：用法同allow，表示禁止特定客户端和地址访问

注意：nginx在解析时，会依次解析，直到遇到自己匹配的才会停止，如下所示，192.168.1.0/24客户端可以访问。
```
location / {
    deny 192.168.1.1;       # 禁止访问
    allow 192.168.1.0;      # 允许访问
    deny all;               # 禁止所有
}
```

## 三 基于密码配置Nginx访问权限

Nginx还支持基于HTTPBasic Authentication协议的认证。该协议是一种HTTP性质的认证办法，需要识别用户名和密码，认证书黑白的客户端不拥有访问Nginx服务器的权限，该功能由ngx_http_auth_basic_module模块支持。

```
auth_basic string | off;     # string是开启认证，并配置验证时的指示信息，off 为关闭
auth_basic_user_file file;   # 用于设置包含用户名和密码信息的绝对文件路径

# 密码文件格式：
name1:password1
name2:password2:comment
name3:password

# 密码支持crypt()函数加密，Linux上使用htpasswd命令
# htpasswd -c -d /nginx/conf/pass_file username
```