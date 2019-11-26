## 一 Rewrite

##### 1.1 Rewrite简介

Rewrite是Web服务器必备的重要基本功能，用于实现URL重写，Nginx的该功能依赖于PCRE。

URL重写的功能：
- 改变网站结构后，客户端无须变动
- 提高安全性

地址重写与地址转发的区别：
- 重写：访问google.cn最终会变成访问goodle.com的过程就是重定向
- 转发：从一个域名到另外一个已有站点的访问过程，转发后客户端地址访问地址是不变的

#### 2.1 常见指令

## 二 虚拟主机中的location

#### 2.1 location简介

location会根据uri进行不同的定位。语法可以分为三类：
- location = patt {}    # 精准匹配，匹配成功立则停止搜索，立即处理请求
- location patt {}      # 一般匹配
- location  ~ patt {}   # 正则匹配
- location  ^~ patt {}   # 正则匹配，匹配成功后立即处理请求，而不再使用location块中的正则匹配，比如转码的地址/html/%20/data，在遇到配置`^~/html//data`时匹配成功

匹配顺序：首先看有没有精准匹配，如果有，则立即解析，停止继续查找，没有则依次按照一般匹配，正则匹配类推。  

```
# 精准匹配
location = / {
    root    html        
    index   index1.html
}

# 一般匹配
location /index.html {
    root    html
    index   index2.html
}

# 正则匹配
location ~ image {
    root    /image
    index   index3.html
}
```

#### 2.2 rewrite 重写

重写常用命令：
```
if (条件) {}    # 注意空格不能少
set             # 设置变量
return          # 返回状态码
break           # 跳出rewrite
rewrite         # 正式重写
alias           # 重新指定目录
```

条件写法：
- = 判断相等，用于字符串比较
- ~ 用来正则匹配，不区分大小写
- ~*用来正则匹配，区分大小写
- -f -d -e 依次判断是否为文件、是否为目录、是否存在

```
# 案例一：禁止某个ip访问
location / {
    if ($remote_addr = 192.168.1.100) {
        return 403
    }
    root html;
    index index.html;
}

# 案例二：正则匹配-不允许IE访问
location / {
    if ($http_user_agent ~ MSIE) {
        rewrite ^.*$ 503.html;
        break;                          # 没有break会循环重定向
    }
    root html;
    index index.html;
}
```

if判断：
```
if ($slow) {
    # 在这里设置nginx配置
}

if ($request_method) == POST {              
    return 405;
}

# 变量与正则之间使用四种连接方式： ~（区分大小写） ~*（不区分大小写） !~ !~*  （这2个匹配失败认为条件是true）

if ($http_user_agent ~ MSIE){}              # 如果$http_user_agent中包含 MSIE则会true

if ($http_cookie ~* "id=([^;]+)(?:;|$)"){}  # 使用$1和$2截取获取到的值

 # 判断请求的文件是否存在,!-f表示是否不存在，-d和!-d用来判断目录，-e和!-e用来同时判断文件或目录，-x和!-x表示判断是否可执行
if (-f $request_filename) {}               
```

break:当满足break之前的匹配，则跳出当前域
```
location / {
    if ($slow) {
        set $id $1
        break;
        limit_rate 10k;
    }
}
```

return:用于完成对请求的处理，该指令后的配置是无效的
```
return [text]               # text为响应体内容
return code URL;            # code为状态码，0-99，URL为响应地址
return URL;
```

rewrite：重写
```
server {
    listen 80;
    server_name jump.test.com;
    rewrite ^/ http://www.test.info/            # 域名跳转
}
```
