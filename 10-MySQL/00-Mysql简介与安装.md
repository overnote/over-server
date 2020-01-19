## 一 MySQL简介

MySQL是当前使用最广泛的开源关系型数据库。  

Mysql版本分类：
- 社区版：免费，加如了很多未经严格测试的新特性，没有官方技术支持
- 商业版：收费，包含了一些额外的组件，如：更好的企业级备份，更好的高可用功能，更好的可扩展功能，安全功能，企业级监控/审计等，并且拥有完整的官方技术支持

## 二 MySQL安装

#### 2.0 安装贴士

Mysql下载地址：https://dev.mysql.com/downloads/mysql/    

贴士：
- 推荐1：学习推荐下载5.7版本。
- 推荐2：选择Win版本可以点击图片，进入Installer MSI方式

Linux安装的三种方式：
- 安装包方式安装：最简单的二进制安装方式，如使用rpm，deb安装下载的安装包。
- 二进制安装：普通的二进制安装方式，适合快速部署，因为是已经编译好的文件，安装文件体积较大，安装后会提供默认的mysql目录
- 源码安装：高级运维安装方式，需要编译，且可以自定义配置和模块

Linux安装原则：  
- 源码安装虽然提供了高度定制性，但是当前的互联网企业都是集群架构，上百集群中使用源码方式安装mysql显然压力很大，推荐使用二进制方式安装
- 有定制需求的企业，可以在源码定制完成后，制作专属的二进制包进行安装

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

#### 2.1.1 安装包方式安装

这里以CentOS系统为介绍，但是由于Yum源版本，较低，可以前往上述网址下载对应Mysql版本后，执行本地安装：
```
yum install 安装包名           # 也可以使用 rpm -ivh 安装包名
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

#### 2.1.2 yum源安装

地址： https://dev.mysql.com/downloads/repo/yum/  

下载yum源后，执行源安装：
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

#### 2.2  CentOS7 二进制方式安装 mysql5.7

0 准备工作
```
# 关闭防火墙
systemctl stop firewalld.service        # 停止firewall
systemctl disable firewalld.service     # 禁止firewall开机启动
firewall-cmd --state                    # 查看默认防火墙状态,notrunning为关闭

# 删除mysql，mariadb
rpm -qa|grep mariadb
yum remove mariadb*
rpm -qa|grep mysql
yum remove mysql*
rm /etc/my.cnf   
rm /etc/init.d/mysqld                              
```

1 下载并上传安装包
```
选择：MySQL Community Server 5.7 -> Linux-Generic 64 -> Compressed TAR Archive
或者：wget https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.25-linux-glibc2.12-x86_64.tar.gz
```

2 解压
```
tar zxvf mysql-5.7.25-linux-glibc2.12-x86_64.tar.gz
mv mysql-5.7.25-linux-glibc2.12-x86_64 /usr/local/mysql
```

3 创建用户
```
groupadd mysql
useradd -r -g mysql -s /bin/false mysql      # 创建一个不可登陆的mysql用户
```

4 创建数据目录
```
cd /usr/local/mysql
mkdir data
chown -R mysql.mysql .
```

5 初始化数据库，此时会生成临时密码，需要记录下来
```
/usr/local/mysql/bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
```
注意：此时会生成一个临时密码，需要记录

6 建立配置
```
vim /etc/my.cnf
# 添加内容
[mysqld]
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
```

7 启动mysql
```
# 启动方式一（不推荐） 手动启动
/usr/local/mysql/bin/mysqld_safe --user=mysql &

# 启动方式二（推荐） 并设置为开机启动
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
chmod a+x /etc/init.d/mysqld                        
chkconfig --add /etc/init.d/mysqld
chkconfig mysqld on
service mysqld start                                # 如果之前启动过，使用 restart
```

8 本地连接
```
/usr/local/mysql/bin/mysql -uroot -p                # 输入刚才保存的密码
```

9 重置密码
```
# 本地连接后执行：
alter user root@'localhost' identified by '(test123)'; # 密码被修改为：(test123)
```

10 授权远程访问
```
grant all privileges on *.* to root@'%' identified by '123456';
flush privileges;
```
解释：
- `*.*`分别代表所有的数据库名和所有的数据库表
- root@’%’中的root代表用户名，%代表ip地址，%也可以指定具体的ip，如：root@localhost,root@192.168.10.129
- flush是授权刷新命令；

#### 2.3.1 CentOS7 源码方式安装 mysql5.7

选择源码包方式，无非是为了进行一些定制，比如：精简mysql，优化参数，自定义安装位置。但是编译安装非常麻烦，笔者推荐下载sourcecode后，编译为一个rpm包后，再执行rpm安装。

本节使用源码rmp包安装。  



#### 2.3.2 直接源码方式安装 mysql5.7

彻底的源码安装： 
https://www.cnblogs.com/yangchunlong/p/8477743.html