## 一 有序集合ZSET简介

有序集合ZSET同样存储着键值对，有序集合的键称为成员，每个成员是各不相同的，而集合的值被称为分值（score，浮点数）。   

与set不同的是每个元素都会关联一个double类型的score，表示权重，通过权重将元素从小到大排序，所以zset可以随意调整位置，通过更改score。元素的score可以相同。  

由于有序集合使用散列表和跳跃表实现，读取中间的数据速度也很快，时间复杂度是O(log(N))。  

`list set zset`区别:
- list：有顺序的链表，元素不能重复，存储的元素按照我们自己添加的顺序排序；
- set：没顺序的集合，元素可以重复，不依赖于添加、大小顺序去，且值可以重复；
- zset：有顺序的集合，元素不能重复，根据权重大小从小到大排列；

## 二 ZSET常用命令
```
# 增加
ZADD key score member [score memeber...]            # 添加元素并记录成员分值，如果元素已存在，则修改分数为新分数，分数可以是正数、小数。
                                                      ZADD test +inf c	+inf -inf分别表示正无穷和负无穷。
# 删除
ZREM key member [member...]                         # 从集合中移除指定成员，返回被移除成员数量

# 获取
ZRANGE key start stop				                # 按照分数大小，返回指定范围内的元素，可选参数是 
                                                        withscores 不但返回元素，还会返回对应的分数
ZREVRANGE key start stop                            # 按照分数从大到小排列，如果元素分数相同，按照0-9-A-Z-a-z排序，中文按照UTF-8编码排序。
ZCARD key                                           # 返回集合中成员数量

# 其他
ZINCRBY key increment member                        # 将member分值增加increment，返回值是更改后的分数，如果不存在该元素，则先建立，并分数默认赋值为0
ZCOUNT key min max                                  # 返回介于min和max之间的成员数量
ZRANK key member                                    # 返回成员memeber在有序集合中的排名
ZSCORE key memeber                                  # 返回成员的member分值
ZRANGE key start end [WITHSCORES]                   # 返回有序集合中排名介于start和end之间的成员，如果给定了可选项，那么命令会将成员的分值也一并返回

# 范围类型数据操作
ZREVRANK key member                                             # 返回有序集合成员member的排名，按照分值从大到小
ZREVRANGE key start end [WITHSCORES]                            # 返回有序集合给定排名范围内的成员，按照分值从大到小
ZRANGBYSCORE key min max [WITHSCORES] [LIMIT offset count]      # 返回有序集合中分值介于min和max之间的所有成员，
                                                                    如果不希望包含两端的元素使用(min max)，位置支持-inf和+inf
ZREVRANGBYSCORE key max in [WITHSCORES] [LIMIT offset count]    # 返回有序集合中分值介于min和max之间的所有成员，并按照分值从大到小返回    
ZREMRANGBYRANK key start end                                    # 移除有序集合中排名介于start和end之间的所有成员
ZREMRANGEBYSCORE key min max                                    # 移除有序集合中分值介于min和max之间的所有成员

# 集合间操作
ZINTERSTORE dest-key count key [key...]                         # 交集 
ZUNIONSTORE dest-key count ke [key...]                          # 并集    

```