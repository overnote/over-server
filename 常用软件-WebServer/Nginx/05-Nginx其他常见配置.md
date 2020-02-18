## 一 gzip压缩

Gzip压缩依赖于模块nginx_http_gzip_module，常见指令有：
```
gzip    on|off;                     # gzip功能的开启与关闭，默认为off
gzip_buffers number size;           # number申请系统缓存空间个数，size是每个缓存空间大小，默认number*size=128，即 32 4k | 16 8k;
gzip_comp_level level;              # 压缩程度，1-9级，程度越高，压缩率越高，但是效率也越低
gzip_disable regex ...;             # 根据regex正则配置不启用gzip的客户端种类，如 MSIE[4-6]\
gzip_http_version 1.0|1.1;          # 默认值为1.1,即客户端使用1.1及其以上版本时才支持gzip
gzip_min_length length;             # 文件越小，压缩效果反而不好，这里设置最小大小，默认为20，建议设置为1024
gzip_proxied                        # 反向代理中设置是否对后端返回数据进行gzip压缩
```

## 二 自动列目录

如果当前目录下不存在index指令设置的默认首页文件，那么此时可以自动列出文件目录，需要配置如下：
```
locathon / {
    autoindex on;
}
```

其他相关设置
```
autoindex_exact_size [on|off]   # 设置索引时文件大小（B，KB,MB,GB）
autoindex_localtime [on|off]    # 设置索引时文件时间，默认为关闭（GMT时间）
```

## 三 浏览器本地缓存

在配置文件中的，http，server，location中都可以针对上述作用于进行浏览器缓存配置。
```
# expires off           # 
expires [time|epoch|max|off]          # time
```

默认是关闭缓存的，即off，time值可以是正数、负数，epoch为1970年Jan1开始，max指定为2037年年底。值为-1表示永远过期。

## 四 Nginx与Java的Tomcat配合

静态文件可以交给nginx处理，而动态文件如.jso,.do可以由Nginx反向代理到Tomcat的服务器处理：

```
http {
    ...

    upstream tomcat {
        server 127.0.0.1:8000;
    }

    server {
        listen 80;
        servername www.test.com;
        index index.html index.jsp index.do default.jsp default.do;
        root /data/www;
        ...

        location ~ \.(jsp|jspx|do)?$ {
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_pass http://tomcat;
        }

        ...
    }

    ...
}
```

## 五 https支持

https的支持需要nginx在安装时加入ssl模块：
```
./configure –with-http_ssl_module
```

配置如下：
```
server {

    ...

    listen 443;
    ssl on;
    ssl_certificate ******.pem;
    ssl_certificate_key ******.key;

    ...
}

```

## 六 防盗链

Nginx有三种方法可以进行防盗链：
- 对Nginx下所有项目的指定资源不同文件类型进行防盗链，比如对gif、jpg、png、swf、flv、mp3、mp4等资源进行防盗链
- 对指定目录或者指定项目目录进行防盗链，比如Nginx下有3个项目，A、B、C，可以对A目录下的images进行防盗链，也可以对B目录下的images进行防盗链
- nginx的第三方模块ngx_http_accesskey_module 可以实现下载文件的防盗链


对指定文件防盗链：
```
location ~* \.(gif|jpg|png|swf|flv)$ { 
  valid_referers none blocked my.cn; # my.cn是可以盗链的域名或IP，一般情况可以把google，baidu等加入
  if ($invalid_referer) {            # 不推荐这样做：这样会不断地302重定向很多次，会加重服务器的负担，除非有单独图片服务器
    # 如果有人非法盗链资源，则返回一张防盗链的图片
    # rewrite ^/ https://www.my.cn:90/picture/images/details-image-1.jpg;    
    return 403; 
  } 
}
```

对指定目录进行防盗链：
```
location /img/ {  
  valid_referers none blocked server_names my.cn ;  # 允许访问该目录的域名或IP
  if ($invalid_referer) {return 403;}               # 不允许访问返回403
}
```