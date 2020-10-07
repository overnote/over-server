## 一 MySQL权限机制

在MySQL默认的数据库中有一个默认数据库mysql，存储了整个数据库的管理信息，包括账户与权限信息，主要位于user表、db表、host表。     

在执行sql指令时，MySQL会从user表开始，逐级下查，判断用户、主机是否对表、列等等具备执行条件。  
- user表：存储账户信息，进行全局的权限控制，如某个账户是否具备insert权限
- db表：记录用户对数据库的操作权限
- host表：记录某个主机对数据库的操作权限，配合db表可以做更加细致的控制
- tables_priv表：设置用户对表的权限
- columns_priv表：设置用户对列的权限。注意：列级别的权限是通过视图实现的。

## 二 账户管理

### 2.1 新增用户

可以通过sql语句的方式来创建用户，但是由于user表字段太多，这样做很容易出错，也很麻烦，一般使用如下的命令：
```sql
# 方式一（也不常用）：创建一个无权限的空白用户
create user 'test1'@localhost identified by '1234';
# 方式二：授权方式，用户不存在则直接创建
grant all on db1.* to 'lisi'@'192.168.52.128' identified by '1234';     # 给用户lisi授权db1数据库的所有权限
flush privileges;              # 手动刷新缓存，因为mysql是基于缓存的，不刷新，则新建用户mysql无法获取到
```
贴士：
- 授权的时候一般遵循权限最小化原则，不应该授权all，如果只是要授权查看、新增数据，则`grant select,insert on ...`  
- **mysql中的用户是包括用户名与来源主机的**
- 除了select等，还有个`USEAGE`权限，使用`create`语句创建的用户就会具备该权限，表示空权限！
### 2.2 删除用户

```sql
drop user 'username'@'host';
```

### 2.3 查看用户权限

```sql
show grants for 'username'@'host';
```

### 2.4 收回权限

revoke语句与grant语句相似：
```sql
revoke all|grant option on db1.* from 'lisi'@'192.168.52.128' identified by '1234';
```

## 三 密码修改

通用语句：
```sql
set password=password('1234');
```

如果登录的是root用户，可以给别的用户修改密码：
```sql
set password for 'user'@'host'=password('1234');
```

也可以使用update语句修改mysql库的user表。  

也可以使用mysqladmin下发命令修改，此时无需登录mysql内部，直接在操作系统中执行：
```sql
mysqladmin -ulisi -p1234 password "5678"
```

## 三 忘记root密码

```sql
# 第一步：关闭数据库服务
service mysql stop

# 第二步：无密码启动数据库
mysqld_safe --skip-grant-tables & 
# 这里一般需要创建如下目录
mkdir /var/run/mysqld
chown mysql. /var/run/mysqld
touch /var/run/mysqld/mysqld.pid
chown mysql. /var/run/mysqld/mysqld.pid

# 第三步：使用空密码进入数据库（mysql命令后直接回车）
mysql

# 第四步：使用update语句修改root密码
use mysql
update user set password=password('1234') where user='root'
exit

# 第五步：关闭数据库并重新以正常方式启动
service mysql restart       # 如果无法重启，则可以kill后启动：service mysql start 
```