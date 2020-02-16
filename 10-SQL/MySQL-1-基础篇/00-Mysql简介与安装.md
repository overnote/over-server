## 一 MySQL简介

MySQL是当前使用最广泛的开源关系型数据库。  

Mysql版本分类：
- 社区版：免费，加如了很多未经严格测试的新特性，没有官方技术支持
- 商业版：收费，包含了一些额外的组件，如：更好的企业级备份，更好的高可用功能，更好的可扩展功能，安全功能，企业级监控/审计等，并且拥有完整的官方技术支持

## 二 MySQL安装

### 2.0 安装贴士

Mysql下载地址：https://dev.mysql.com/downloads/mysql/    

贴士：推荐下载安装5.7版本，如果要安装win版本，则可以点击上述网址中的图片，进入Installer MSI方式。  

登录命令：
```
# 简要输入，直接进入当前host下的mysql，并以root登录，然后会提示输入密码
mysql -p

# 登录
mysql -h192.168.0.1 -P3306  -uroot -p123456
```

导入数据文件：
```
# 将back.sql文件批量导入数据库
mysql < back.sql            
```

### 2.1.1 Linux直接安装编译好的二进制包

直接下载编译好的二进制包，然后使用Linux对应的安装工具进行安装即可，这里以CentOS系统为例：
```
rpm -ivh 安装包名   # 也可以使用 yum install 安装包名 
```

启动mysql（两种方式作用一样）：
- `service mysql start`
- `/etc/init.d/mysql start`

启动后，由于是第一次安装，密码是随机的，建议再执行一次安全安装命令:
```
# 密码位于 .mysql_secret
mysql_secure_installation                      

# 执行后的一些提示 Remove anonymous users？ 是否移除默认用户，mysql安装后存在一个可以查看数据的默认用户，这里选择移除

# 执行后的一些提示 Disallow root login remotely？ 是否允许root用户远程登陆，一般不运训
```

### 2.1.2 Linux使用镜像源安装

使用地址：`https://dev.mysql.com/downloads/repo/yum/`下载yum源后，执行源安装：
```
rpm -ivh 源包名     # 会将对应的源版本同步到 /etc/yum.repos.d 文件中
```

安装mysql：
```
# 查看有哪些源
yum list|grep MySQL   
# 安装mysql  
yum install MySQL-client MySQL-server
```