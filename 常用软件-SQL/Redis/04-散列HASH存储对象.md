## 一 散列HASH简介

散列可以存储多个键值对之间的映射，可以把散列想象为一个猥琐版的Redis或者类似JSON。 

hash一般可以理解为用来存储对象，将对象的键值一起做了存储。一个散列类型的键可以包含至多2的32次方-1个字段。  

注意：字段值只能是字符串。同样其他数据类型也不支持数据类型嵌套，比如集合类型的每个元素都只能是字符串，不能是另一个集合或者散列。  

散列HASH常用于存储用户信息，如uid与token。

## 二 常用命令 

```
# 设置
HSET key field value                # 设置单个属性：hset person name 'lisi'
HMSET key field value field value.. # 设置多个属性
HSETNX	key field	value			# 与HSET一致，但是如果字段存在，则不执行任何操

# 获取
HGET key field                      # 获取一个属性的值
HMGET key field field ...			# 获取多个属性的值
HGETALL key                         # 获取所有属性和值
HLEN key                            # 返回包含属性的个数							
HKEYS key						    # 获取所有属性
HVALS key						    # 获取所有属性的值

# 删除
HDEL key filed [filed]...           # 如果给定键存在,那么移除这个键,返回值是删除的字段个数

# 散列高级特性
HEXISTS key field                   # 判断属性是否存在，存在返回1，否则返回0，键不存在也返回0
HINCRBY key field increment         # 将field值增加整数increment
HINCRBYFLOAT key field increment    # 将field值增加浮点数increment
# 案例
hincrby person score 60             # 不存在score，则创建，且执行命令前的值为0，命令返回值是增值后的字段值。
```

注意事项：
- 如果散列非常大，可以先使用HKEYS获取所有的键，然后通过只获取必要的值来减少需要传输的数据量。
- HSET命令不区分插入和更新，修改数据时无需事先判断字段是否存在来决定是更新还是插入。执行插入返回1，执行更新返回0，当然如果键不存在，则会创建键。
注意：在redis中，键的类型和命令有关，SET创建的键是字符串类型，HSET命令创建的hash类型，如果使用一种数据类型的命令操作另外一个数据类型的键，会提示错误：
`ERR Operation against a key holding the wrong kind of value`

