## 一 虚拟主机配置

#### 1.0 Nginx配置虚拟主机的三种方式

虚拟主机，在web服务里是一个独立的网站站点，该站点对应独立的域名或者ip或者端口，具备独立的程序和资源目录，能独立的对外提供服务供用户访问。  

- 基于域名的虚拟主机：通过域名区分虚拟主机，常用来区分公司不同的二级域名
- 基于端口的虚拟主机：通过端口区分虚拟主机，长用来做公司内部网站，后台系统
- 基于IP的虚拟主机：通过ip区分虚拟主机，基本不用，不支持ifconfig别名，配置文件可以。

#### 1.1 基于域名的虚拟主机

```
server {
    listen 80;                      # 监听的端口
    server_name www.test1.com;      # 监听的域名：当用户访问 test1.com时候该server生效
    location / {
        root    html/test1;         # 默认文件夹，也可以是相对路径(相对于/usr/local/nginx)
        index   index.html;         # 默认取哪个页面
    }
}
server {
    listen 80;                     
    server_name bbs.test1.com;         
    location / {
        root    html/bbs;             
        index   index.html;        
    }
}
```

注意：配置完毕后，修改本地host为 nginx所在服务器主机地址 test1.com 

#### 1.2 基于端口的虚拟主机

```
server {
    listen 3000;                    # 访问www.test1.com:3000时生效
    server_name www.test1.com;      
    location / {
        root    html/test1;           
        index   index.html;      
    }
}
```

#### 1.3 基于IP的虚拟主机

```
server {
    listen 3000;                 
    server_name 192.168.1.200;      
    location / {
        root    html/test1;           
        index   html/index.html;      
    }
}
```

#### 1.4 虚拟主机别名
```
server {
    listen 80;                     
    server_name www.test1.com test1.com;        # 此时访问带或者不带www都可以进入该目录  
    location / {
        root    html/test;             
        index   index.html;        
    }
}
```

#### 1.5 虚拟主机写法优化

通常一个网站拥有多个二级域名，且拥有多个独立的后台，那么就需要配置多个虚拟主机（server），此时会造成nginx.conf文件越来越大，可以将不同的server进行拆分：
```
http {                                      
    include    mime.types;                 
    default_type   application/octet-stream;  
    sendfile   on;
    keepalive_timeout  65;
    include    extra/www.conf;
    include    extra/bbs.conf;
    include    extra/pms.conf;
}
```