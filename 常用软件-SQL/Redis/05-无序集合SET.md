## 一 集合SET简介

集合SET通过散列保证存储的元素（即每个字符串）都是各不相同的，由于采用无序方式存储，集合SET不能插入元素到两端。 

元素为string类型，同样存储内容最多不能超过2的32次方-1个。  

与列表不同的是：元素具有唯一性，不重复。  

集合在redis内部是使用值为空的散列表实现，所以操作的复杂度为O（1），非常便于多个集合类型键之间进行并集、交集、差集运算。  

实际应用场景：文章的标签都是不相同的，且展示这些标签的时候排序没有要求，那么可以使用集合存储标签。

## 二 集合SET常用命令
```
# 增加
SADD key item [item...]                 # 将给定元素添加到集合，返回被添加的元素数(不包括重复的)，键不存在则创建，存在则忽略

# 删除
SREM key item [item...]			        # 删除元素，返回成功删除的个数
SPOP key							    # 从列表左边删除一个元素，返回该被删除元素，由于set无序，会随机删除

# 获取
SMEMBERS key					        # 返回key集合所有的元素
SCARD key						        # 返回集合元素个数

# 其他
SISMEMBER key item                      # 检查指定元素是否存在于集合,存在返回1，不存在返回0，由于是个O（1）操作，无论集合数目有多少，都能很快返回结果
SRANDMEMBER key [count]                 # 从集合里随机返回一个或者多个元素，当count为正数，返回的随机元素不会重复，count为负数可能重复
SMOVE key source-key dest-key item      # 如果集合source-key包含元素item，那么移除该元素并添加到dest-key中，移除成功返回1，否则返回0

# 多集合命令
SDIFF key [key...]                      # 差集：返回存在于第一个集合，而不存在于其他集合中的元素
SDIFFSTORE dest-key key [key...]        # 差集：返回存在于第一个集合，而不存在于其他集合中的元素，并存储到dest-key
SINTER key [key...]                     # 交集：返回同时存在于所有集合中的元素
SINTERSTORE dest-key key [key...]       # 交集：返回同时存在于所有集合中的元素，并存储到dest-key
SUNION key [key...]                     # 并集：返回至少存在于一个集合的元素
SUNIONSTORE dest-key key [key...]       # 并集：返回至少存在于一个集合的元素，并存储到dest-key
```