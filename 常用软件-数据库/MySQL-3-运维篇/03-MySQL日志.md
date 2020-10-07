## 一 错误日志

### 1.1 错误日志配置

错误日志记录mysql的运行状态，默认文件名为hostname.err（5.6的RPM包安装时位于/var/log/myqld.log），错误日志文件的配置位于`/etc/my.cnf`：
```
[mysqld_safe]
log-error=/var/log/mysqld.log
```

贴士：修改了mysql配置文件后，需要重启mysql服务，配置文件才能生效。  

删除日志的办法：`echo > /var/log/mysqld.log`

### 1.2 错误码

===TODO====

### 1.3 通用查询日志

通用查询日志默认是不开启的，只有需要采样分析时收工开启：
```sql
# 方式一：使用命令
set global general_log=1;       # 设置为0则是关闭
# 方式二：修改配置，不推荐！
[mysqld]
general-log-file[=[ath/[filename]]]
general-log=1
```

采样日志一般用于分析mysql哪些列，哪些行等被访问的频繁，通用查询日志非常影响性能、空间。  

通用查询日志默认位于日志目录中，文件名以主机名开头。  

## 二 慢查询日志

慢查询日志用于查询缓慢的SQL语句，同样默认不会开启，也是需要进行采样分析的时候才会手工开启。  

慢查询的开启参数很多，且慢查询可能需要采样的时间较长，一般推荐使用修改my.cnf的配置（需要重启），常用参数有：
```
[mysqld]
slow_query_log=on|off               # 是否开启慢查询
slow_query_log_file = file          # 慢查询日志文件，默认为hostname-slow.log
long_query_time=2                   # 指定多少秒未返回结果的语句为慢查询
long-queries-not-using-indexes      # 记录所有没有使用索引的查询语句
min_exmained_row_limit=1000         # 记录查找了多于1000次引发的慢查询
log-slow-admin-staements            # 记录慢的OPTIMIZE TABLE，ANALYZE TABLE 和 ALTER TABLE语句
log-slow-slave-statements           # 记录由slave所产生的慢查询
```

贴士：**mysql配置的 _  和 - 大多数场景中效果是一样的**  

如果需要临时查看，也可以使用直接设置的方式启用慢查询，此时无需重启：
```sql
set @@global.slow_query_log=1       # 为0则关闭
set @@global.long_query_time=3
```

## 三 二进制日志


