## 一 字符串STRING简介

Redis的字符串是由字节组成的序列，可以存储：字节串（byte string），整数取值范围取决于操作系统位数，整数也能转换为浮点数，浮点数取值范围和精度则与IEEE754标准的 双精度浮点数一致。

常用命令：
```
# 设置键值
SET key value                   # 设置key的值为value，成功则返回OK，key存在则替换，不存在则创建并设置
SETEX key seconds value			# 设置键值及过期时间，以秒为单位
MSET key value [key value ...]	# 设置多个键值

# 获取键值
GET key                         # 获取key的值，成功则返回键值，不存在该key返回nil，
MGET key1 key2 ...				# 根据多个键获取多个值
MSET k1 v1 [k2,v2...]			# 设置多个键值

# 删除键值
DEL key                         # 删除key，返回被删除的key数量

# 获取长度
STRLEN key						# 获取值长度

# 自增自减:如果不存在则会创建一个值为0的key，并同时进行自增自减。value为空的键执行自增自减，默认为0，如果value是无法解析为十进制数，则返回错误
INCR key                        # key的值自增1
INCRBY key amount               # key值增加amount
INCRBYFLOAT key amount          # key值加上浮点数amount(redis2.6以上支持)
DECR key                        # key值自减1
DECRBY key amount               # key值减少amount
注意：redis的操作是原子性的，即：当2个客户端同时读取到了值为1的一个key，并对该key执行了incr操作，虽然会执行2次操作，但是结果将会是2，不是3。
贴士：对每一类对象使用名为 对象类型（复数形式）:count的键，如：users:count 来存储当前类型对象的数量，每增加一个新对象时都使用incr命令递增该键的值。
由于incr命令建立的键初始值是1，那么很容易得到对象总数，且该总数也是最后一个新增对象的ID。

# 常见处理子串命令
APPEND key value                # 将value追加到key值末尾，键不存在则创建。返回值为总长度。
GETRANGE start end              # 获取范围内字符串
SETRANGE offset value           # 将偏移量offset开始的子串设置为value
```

## 二 位操作

一个字节由8个二进制位组成，redis提供了4个命令可以直接对二进制位操作。比如使用redis设置`set foo bar`后，存储结构如下：
```
b			a			r
0110 0010	01100001	01110010

GETBIT foo 0			    # 返回结果0
GETBIT foo 6			    # 返回结果1

SETBIT foo 0 1			    # 设置第0位为1
```
注意：
- 如果索引长度超过了值得二进制位长度，返回默认值0
- 如果设置的位置超过了二进制位长度，则将中间的二进制位设置为0，如果设置了不存在的键的二进制位的值，则前面的位赋值为0

位运算命令BITOP，支持：`AND OR XOR NOT`运算:
```
SET foo1 bar
SET foo2 aar
BITOP OR res foo1 foo2      # foo1 OR foo2 的值后存储在键res中

BITOPS foo 1			    # 获得键的第一个位值为1的偏移量
BITOPS foo 1 1 2		    # 第二个第三个参数分别指定要查询的起始字节和结束字节，该案例要查询第二个字节到第三个字节之间出现的第一个位值为1的二进制位偏移量
```
注意：如果不设置结束字节且键值的所有二进制位都是1，当要查询值为0的二进制位偏移量时，返回结果是键值长度的下一个字位的偏移量。这是因为redis会认为键值长度之后的二进制位都是0。  

使用位运算的好处：记录用户的性别，0 1 ，那么100万个用户只会占用100kb。 

注意：SETBIT时，如果当前键的值长度小于要设置的二进制位的偏移量时，redis会自动分配内存并将键值得当前长度到制定的偏移量之间的二进制位都设置为0。如果要分配的内存过大，则可能会造成服务器阻塞而无法接受同一时间其他请求。举例：如果网站用户的ID是从100000001开始，那么会造成10多M的浪费，正确做法是给每个用户减去100000001再存储。