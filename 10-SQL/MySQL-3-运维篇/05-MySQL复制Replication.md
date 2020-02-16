## 一 MySQL Replication 简介

Replication可以实现将数据从一台数据库服务器（master）复制到另外一到多台数据库服务器（slave）。  

默认情况下，MySQL的复制机制是异步的，因此无需维持长连接，如果是同步复制，则会极度损耗性能。    

MySQL的复制支持通过配置，直接实现复制所有的库、某些库，甚至库中的某几个表！  

## 二 MySQL Replication 原理

MySQL Replication原理：
- Master将数据的改变写入日志，Slave同步这些日志，并根据日志进行数据的操作！！   
- Slave是通过MySQL连接登录到了Master来读取日志的

贴士：MySQL的复制机制也可以用来实现数据的备份，但是该备份方案无法应对SQL语句的错误执行！！！（比如master执行的删库操作，slave也会执行！）

## 三 Replication的配置

### 3.1 最简单的双机主从复制

现在又Master和Slave主从两台机器，配置其复制的步骤如下：
```
# 第一步：授权Slave，由于Slave需要登录Master读取日志文件,所以需要给Slave配置权限
mysql>GRANT REPLICATION SLAVE ON *.* TO '登录用户名'@'slave服务器的ip' IDENTIFIED BY 'slave服务器的密码';
mysql>FLUSH PRIVILEGES;

# 第二步：在Master上打开日志，并标识server-id
vim /etc/my.cnf

[mysqld]
log-bin
binlog-format=row
sync-binlog=1
server-id=1

service mysql restart

# 第三步：制作一个Master的完整备份，并且执行prepare。因为传输二进制会消耗很多资源，与其同步，不如先备份，用以节省时间
cd 备份目录
innobackupex --user=备份用户用户名 --password=备份用户密码 /var/lib/backup/     # 后面的目录是备份目录
innobackupex --use-memory=500m --apply-log /var/lib/backup/2020-01-01_13-30-30-05/

# 第四步：将备份copy到slave上
# master:
scp -r /var/lib/backup/2020-01-01_13-30-30-05/SLAVE的ip:/var/lib/backup/
# slave:
cd /var/lib/backup/2020-01-01_13-30-30-05/
mv * /var/lib/mysql
chmod -R mysql.mysql /var/lib/mysql
```