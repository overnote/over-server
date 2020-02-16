## 一 企业级MySQL安装规范

Linux安装的两种方式：
- 二进制安装：推荐使用该方式安装
  - 使用安装包：最简单的二进制安装方式，如使用rpm，deb直接安装下载的安装包
  - 二进制方式：普通的二进制安装方式，适合快速部署，因为是已经编译好的文件，安装文件体积较大，安装后会提供默认的mysql目录
- 源码安装：高级运维安装方式，需要编译，且可以自定义配置和模块

Linux安装原则：  
- 源码安装虽然提供了高度定制性，但是当前的互联网企业都是集群架构，上百集群中使用源码方式安装mysql显然压力很大，推荐使用二进制方式安装
- 有定制需求的企业，可以在源码定制完成后，制作专属的二进制包（如rpm包）进行安装


## 二 CentOS7 二进制方式安装 mysql5.7

**0 准备工作**
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