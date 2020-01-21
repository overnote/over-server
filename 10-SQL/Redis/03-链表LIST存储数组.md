## 一 链表LIST简介

链表LIST可以存储多个相同、不同的字符串，可以暂时想象为JS中的数组，经常用来存储一个有序的字符串列表，常用的操作是向列表两端添加元素，或者获得一个列表的片段。  

列表的内部使用双向链表实现，所以向列表两端添加元素的复杂度为O(1)，获取越接近两端的匀速速度越快。  

案例：向一个有几千万元素的列表获取头部、尾部10条记录和向一个有20个元素的列表中获取头部或者尾部的10条记录的速度是一样的。  

使用链表的代价是依据索引获取元素非常慢，因为需要从第一个元素一直查找到索引元素。  

列表的使用场景：社交网站的新鲜事，只需要获取最新内容、记录日志、队列。

## 二 链表常用命令

```
# 设置
LPUSH key value [value value ...]           # 将value插入链表key右端，返回链表元素个数，不存在则创建
LSET key index value		                # 设置指定索引（基于0）的元素值，可以是始计数，如-1表示列表的最后一个元素
LINSERT key BEFORE|AFTER pivot value        # 在一个元素的前|后插入新元素，返回值是插入后个数，首先会在列表中从左往右查找值为pivot的元素，然后插入

# 获取
LRANGE key start end                        # 获取链表start到end上的所有元素值（链表从0计数）
LINDEX key index                            # 获取链表指定位置元素值，没有则返回nil
LLEN key					                # 返回存储在 key 里的list的长度（个数）

# 删除
LPOP                                        # 删除链表最左侧元素，返回被删除元素
LREM key count value		                # 删除列表中前count个值为value的元素，返回值是时间删除元素个数，根据count值不同，LREM执行方式不同。
                                                count > 0 从列表左边开始删除 count 个值为value的元素
                                                count < 0 从列表右边开始删除 |count|个值为value的元素
                                                count = 0	删除所有值为value的元素
LTRIM key start end                         # 修剪列表,保留start-end的内元素，案例：每次写日志希望只保留最近100条日志，在插入日志后`LTRIM logs 0 99`


# 同理还有R开头的命令，表示从右侧开始，比如RPUSH，RPOP



# 将元素从一个列表移动到另一个列表，或者阻塞执行命令的客户端直到有其他客户端给列表添加元素为止
BLPOP key [key...] timeout                   # 从第一个菲康列表中弹出最左端元素，或者在timeout秒内阻塞并等待可弹出元素出现，同理有BRPOP
RPOPLPUSH source-key dest-key                # 从source-key列表弹出最右端元素，然后将这个元素推入dest-key列表最左端，最后返回该元素
BRPOPLPUSH source-key dest-key timeout       # 作用同 RPOPLPUSH ，但是如果source-key为空，那么在timeout秒内阻塞并等待可弹出元素出现
```

实战：当列表类型作为队列时，source 和destination如果相同，RPOPLPUSH 命令会不断的将队尾的元素移动到队首。例如使用该特性实现一个网站监控系统：存储需要监控的网站，程序不断使用RPOPLPUSH，循环取出一个网址来测试可用性，好处是在于在程序执行过程中仍然可以不断的向网址列表中加入新网址，整个系统易扩展，允许多个客户端同时处理队列。