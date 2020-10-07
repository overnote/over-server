## 一 定位有问题的SQL

### 1.1 查看SQL执行频率

查看SQL执行频率：
```
# 查看 order 相关的SQL执行频率，包括查询、修改、增加等
show status like 'order_______';         # 使用了7个占位符，添加global参数可以查看全局 show global status...

# 查看 InnoDB 的操作数量
show global status like 'Innodb_rows_%';
```

### 1.2 定位低效SQL语句

可以通过2种方式定位执行效率低的SQL语句：
- 慢查询日志：用 `--log-slow-queriest[=file_name]`启动时，启动时间超过 long_query_time 秒的SQL语句会被记录到慢查询日志文件
- `show processlist`：该命令可以查看当前mysql运行的线程，包括线程的状态、是否锁表等，与慢查询日志不同，该方式可以实时查看

### 1.3 explain 对慢语句进行分析

使用explain或者desc命令或可以获取MYSQL执行语句时的一些信息，如SELECT语句执行过程中标如何连接、连接的顺序：
```
explain select * from user;
```
