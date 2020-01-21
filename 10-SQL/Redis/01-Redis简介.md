## 一 Redis初识

#### 1.1 Redis简介

Redis（Remote Dictionary Server）是使用C语言编写的开源高性能键值存储数据库。  

Redis基于内存存储，又具备持久化功能，是当前最热门的NoSQL数据库之一，经常用来处理缓存、购物车、消息队列、任务队列。  

Redis提供了多种数据结构，并具备复制、持久化、客户端分片功能。

使用Redis可以很方便的实现每秒处理上百万次请求。  

Redis优势：
- 性能极高：每秒读速度达11万次，写速度8万1千次；
- 数据类型丰富：支持二进制的Strings,Lists,Hashes,Sets等；
- 原子操作：redis的操作都是原子性的，同时支持对几个操作全并后的原子性执行；
- 特性丰富：支持发布订阅、通知、key过期、持久化等特性；

5种数据结构：
| 数据结构 | 结构存储的值 | 结构读写能力 |
| ---- | ---- | ---- |
| STRING | 可以是字符串、整数、浮点数 | 可以对整个字符串或者其中一部分操作，可以对整数进行自增自减操作 |
| LIST | 链表，链表上每个节点都包含一个字符串 | 可以从链表两端插入获取删除元素，可以根据偏移量进行trim，可以读取单个、多个元素，可以根据值查找、移除元素 |
| HASH | 无序散列表，包含键值对 | 可以添加、获取、删除单个键值对，可以获取所有键值对 |
| SET | 无序集合，每个被包含的字符串都是各不相同 | 可以添加、获取、删除单个元素，可以检查元素是否存在，可以计算交集、并集、差集，可以从集合中随机获取元素 |
| ZSET | 有序映射，按照分值排序，字符串与分值有序映射 | 可以添加、获取、删除单个元素，可以根据分值范围或者成员获取元素 |

5种数据结构常见使用场景：
- STRING:常见的k-v缓存
- LIST:最新消息排行、消息队列、关注列表、粉丝列表
- HASH:存户用户信息（能单独修改用户某一属性信息）
- SET:共同好友，共同爱好，统计网站访问IP（唯一性），好友推荐
- ZSET:排行榜，带权重的消息队列

#### 1.2 Redis安装

redis解压后就可以直接使用了（不像Nginx等Linux软件需要configure，redis官方已经配置过了）。
```
# 下载并加压redis，进入redis目录
make                                        # 编译
make test                                   # 如果提示需要tcl库，需要安装tcl
make PREFIX=/usr/local/redis install        # 执行安装
cp redis.conf /usr/local/redis/             # 复制一份配置文件到redis下
```


#### 1.3 启动 

```
cd /usr/local/redis/
./bin/redis-server redis.conf 

# 设置守护进程启动：修改配置文件 daemonize yes
# 设置密码方式启动：修改配置文件 requirepass 密码  （此处注意，行前不能有空格）
```

#### 1.4 连接

```
# 启动连接
./redis-cli
ping                # 得到回复 PONG

# 有密码和IP或者端口限制的连接方式：
redis-cli -h  yourIp-p yourPort  -a youPassword

# 阿里云开放外网访问，除了要开放6379端口外，还要修改redis配置：
bind 127.0.0.1  修改为 bind 0.0.0.0

# 关闭连接
./redis-cli -p 6379 shutdown	关闭redis服务
```

## 二 redis通用操作

注意：redis命令不区分大小写

```
# 设置键
set test 123            # 设置一个值为123的键test

# 查询值
get test                # 得到"123"          

# 查询键名
keys *                  # 查询当前所有的key
keys test               # 使用keys命令进行精确查找key
keys t*                 # 使用keys命令进行模糊查找key
keys t[e|s]st           # 使用keys进行匹配查找key：test,tsst
keys t??t               # 使用keys进行模糊匹配
kyes \x                 # 匹配转移\x

# 删除键
del test                # 删除key，成功返回1，失败返回0
注意：del不支持通配符，可以使用管道方式删除所有以user开头的键：redis-cli del `redis-cli keys “user*”`

# 改键名
rename test test2       # 修改test名为test2，若test2原本存在，则替换
renamenx test test2     # 若test2原本存在，则返回0

# 键有效期
ttl test                # 查询有效期，返回值-1为永久，key不存在也返回-2
expire test 60          # 设置键test有效期为60秒
pexpire test 1000       # 设置毫秒数
pttl test               # 查询毫秒有效期
persist test            # 设置为永久有效

# 选择数据库
select 1                # 选择数据库，redis默认有16个数据库，默认链接0号

# 其他
randomkey               # 随机获取一个key名
exists test             # 查询键是否存在，存在返回1，不存在返回0
type test               # 查询key类型
move test 1             # 将test移动到1号库
```

Redis没有对键命名进行约束，但一般采用如下格式：
```
语法：
对象类型:对象ID:对象属性来命名一个键，

示例：
user:1001:name          # 多个单词使用 . 分隔
```

## 三 缓存选型：redis/memcache

- 支持的数据结构：memcache只支持存储字符串，redis支持多种数据结构
- 持久化：memcache不支持持久化，redis提供了两种持久化方式。持久化能让缓存服务故障能够快速恢复，避免压力集中到数据库，缺点是短期数据不一致
- 高可用：memcache高可用需要二次开发，redis天然支持集群：主动复制、读写分离
- 并发模型：memcache使用多线程，主线程监听，worker子线程处理业务，可能存在锁冲突，redis是单线程，无锁冲突，但是不能利用多核特性，影响整体吞吐量。
- 网络模型：二者都是非阻塞IO复用模型
- 内存分配：memcache使用预分配内存池的方式管理内存，能够省去内存分配时间。redis则是临时申请空间，可能导致碎片。
- 内存使用：memcache把所有的数据存储在物理内存里。redis有自己的VM机制，理论上能够存储比物理内存更多的数据，当数据超量时，会引发swap，把冷数据刷到磁盘上。
