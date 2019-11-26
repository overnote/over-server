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
