## 一 MySQL Replication 简介

Replication可以实现将数据从一台数据库服务器（master）复制到另外一到多台数据库服务器（slave）。通过配置，可以直接实现Mysql的复制，当然也支持只复制一些特定库、特定的表。  

默认情况下，MySQL的复制机制是异步的，因此无需维持长连接，如果是同步复制，则会极度损耗性能。    

## 二 MySQL Replication 原理

MySQL Replication原理：
- Master将数据记录到二进制日志（binary log，位于log-bin中）中，这些记录称为二进制日志事件（binary log events）
- Slave将master的 binary log events 拷贝到它的中继日志（relay log）
- Slave重做中继日志中的事件，进行数据重演

贴士：MySQL的复制机制也可以用来实现数据的备份，但是该备份方案无法应对SQL语句的错误执行！！！（比如master执行的删库操作，slave也会执行！）

## 三 Replication的配置

主库配置其复制的步骤如下：
```
# 第一步：授权Slave，由于Slave需要登录Master读取日志文件，所以需要给Slave配置权限
mysql>GRANT REPLICATION SLAVE ON *.* TO '登录用户名'@'slave服务器的ip' IDENTIFIED BY 'slave服务器的密码';
mysql>FLUSH PRIVILEGES;

# 第二步：在Master配置主从，即serverid

vim /etc/my.cnf
# 开启主从复制，设置二进制文件名
log-bin = mysql-bin
# 指定主库serverid，这里和从库不能一致
server-id=1
# 指定同步的数据库，如果不指定则同步全部数据库
binlog-do-db = my_db
# 指定复制模式
binlog_format=MIXED

# 第三步：执行SQL语句查询状态，会得到当前数据库的一些状态
SHOW MASTER STATUS \G;
```

从库配置如下：
```
vim /etc/my.cnf

# 指定从库serverid，不可与master重复
server-id=2
```

从库中执行以下SQL：
```
# 注意后面2个参数是在MASTER中执行 SHOW MASTER STATUS; 后得到的结果。pos是位置的意思
CHANGE_MASTER TO
master_host='127.0.0.1',
master_user='slave01',
master_password='123456',
master_port=3306
master_log_file='mysal0bin.000006',
master_log_pos=1120;

# 启动slave同步
START SLAVE;

# 查看同步状态
SHOW SLAVE STATUS \G;
```

## 四 MySQL复制模式

使用命令查看mysql复制模式：
```
show global variables like 'binlog%';

# 显示结果中 binlog_format 即是复制模式
```

MySQL的复制模式有：
- 基于SQL语句的复制：简写SBR，每一条会修改数据的sql语句都会记录到binlog中
  - 优点：不需要记录每一条sql语句和每一行数据变化，减少了binlog日志量，节约IO，提高了性能
  - 缺点：可能导致数据不一致，如 `select now()`
- 基于行行复制：默认方式，简称RBR，不记录sql语句，而是记录数据修改的结果，解决了基于SQL复制的缺点，但是会产生大量的日志记录，比如用户表有10万行，每个用户积分+1，则产生了10万个日志！
- 混合复制：简称MBR，一般的复制使用STATEMENT模式保存binlog，对于STATEMENT模式无法复制的操作使用ROW模式保存binlog，MySQL会根据执行的SQL语句选择日志保存方式。  

