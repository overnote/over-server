## 一 group

根据设定的字段将文档分为不同的组，然后将每个组的文档数据进行聚合再返回一个最终的结果文档。 

```
db.collection.group({
    key: {filed: 1},          # 指定按照哪个字段分组
    initial: {count: 0}       # 分组前的初始化，这里的变量可以在reduce中作为result属性
    cond: {}                  # 可选项：查询条件
    reduce: function(c,r){}, # 业务处理函数，c是当前分组中遍历到的文档，r是当前文档 
    keyf: function(doc){},   # key有时候不满足分组，可以把文档中字段重组分组，doc是当前文档
    finnalize: function(result){}
})
```

案例：统计用户user集合中大于13岁的人数
```
# 查询语句
db.user.group({
    key: {age: 1},
    initial: {count: 0},
    cond: {"age": {gt: 13}},
    reduce: function(curr, result){
        result.count += 1;
    },
    finalize: function(result){
        result.countAge = result.age;
        result.countNum = result.count;
    }
})

# 输出结果
[
    {"age":30,"count":1,"countAge":30,"countNum":1}
    ...
]
```

注意：上述group在3.4中已经不推荐使用，推荐使用aggregate聚合挂到中的group策略或者使用mapreduce来实现group。

## 二 MapReduce

MapReduce相当于关系型数据库中的`group by`，使用MapReduce统计需要调用Map函数和Reduce函数，Map函数会调用emit(key,value)方法，此方法可以遍历集合中的所有记录，然后将key和value传递给reduce函数进行处理。  

MapReduce最优秀的地方在于能够依据指定的规则自动分割任务，分发到不同的服务器中执行，每台计算机都完成一部分，然后再讲结果合并为最终结果。 

现实中的案例解释：假如军训时教官需要知道面前才加军训的学生数量。  
```
方案一：派一个学生去一个一个数人数，这种方式类似单机单线程
方案二：派多个学生去数，分别上报，类似单机多线程
方案三：教官指定规则，所有学生按照自己的班级站在一起，每班学生数上报给一个同学，这个同学再对每个班级的报数进行汇总，类似MapReduce。
```
从三种方法可以看出，当人数越来越多时，MapReduce的效率就会越来越高。  
MapReduce主要通过map、shuffle、reduce三个过程来运算，按照班级集中在一起，就是map，在map时，分别要输入key，value，ke是映射规则，即班级名，value就是这个班级的人数，把每个班级的人数收集起来就是shuffle的过程。map的输出之后就到了shuffle流程，该部分由Mongo自动完成，所以我们只需要书写map和reduce即可。shuffle会对map的输出部分进行洗牌，通过key进行分组获得一个List：
![](/images/sql/mongo01.png)
shuffle洗牌后的输出会作为，reduce接着对数据进行业务逻辑操作，即对上述结果进行化简，相加。

```
db.collection.mapReduce(
    function(){emit(key,value)},    # 一个map函数，根据key值把同组value放入values，然后传递给下一个函数
    function(key, values),          # reduce函数，kye是分组字段，values是同组的值
    {
        out: collection 或 {},      # 指定结果集生成地方
        query:{},
        sort:{},
        limit:{},
        finalize:function(key, reduced){}
    }
)
```

使用mapReduce统计不同年龄的人数
```
db.user.mapReduce(
    function(){
        emit(this.age,{age:this.age, count: 1})
    },
    function(key, values){
        var count = 0;
        values.forEach(function(val){
            count += val.count;
        });
        return {age: key, count: count}
    },
    {
        out: {inline: 1},   # 数据保存在内存
        finalize: function(key, reduced){
            return {"年龄": reduced.age,"人数":reduced.count}
        }
    }
)
```

MapReduce擅长处理大数据量的数据，使用MapReduce分布式处理一般是快于单机多线程处理

## 三 聚合、管道、表达式

#### 3.1 聚合管道初识

聚合(aggregate)主要用于计算数据，类似sql中的sum()、avg()。 

```
db.集合名称.aggregate([
    {管道:{表达式}},
    {管道:{表达式}},
    ....
    ])
```

管道在Unix和Linux中一般用于将当前命令的输出结果作为下一个命令的输入，比如如下命令：
```
ps ajx | grep mongo
```

在mongodb中，管道具有同样的作用，文档处理完毕后，通过管道进行下一次处理。  
常用管道：
- $group：将集合中的文档分组，可用于统计结果
- $match：过滤数据，只输出符合条件的文档
- $project：修改输入文档的结构，如重命名、增加、删除字段、创建计算结果
- $sort：将输入文档排序后输出
- $limit：限制聚合管道返回的文档数
- $skip：跳过指定数量的文档，并返回余下的文档
- $unwind：将数组类型的字段进行拆分


常用表达式：
- $sum：计算总和， $sum:1同count表示计数
- $avg：计算平均值
- $min：获取最小值
- $max：获取最大值
- $push：在结果文档中插入值到一个数组中
- $first：根据资源文档的排序获取第一个文档数据
- $last：根据资源文档的排序获取最后一个文档数据

```

示例：统计男生、女生的总人数
db.stu.aggregate([
    {$group:
        {
            _id:'$gender',
            counter:{$sum:1}
        }
    }
])
```
Group by null 可以 将集合中所有文档分为一组
```
示例：求学生总人数、平均年龄
db.stu.aggregate([
    {$group:
        {
            _id:null,
            counter:{$sum:1},
            avgAge:{$avg:'$age'}
        }
    }
])
```

#### 3.2 常用管道操作器

$project  
用于修改输入文档的结构，如重命名、增加、删除字段、创建计算结果  
```
例1：查询学生的姓名、年龄
db.stu.aggregate([
    {$project:{_id:0,name:1,age:1}}         # 0 排除该字段 1保留 也可以直接赋值
])
例2：查询男生、女生人数，输出人数
db.stu.aggregate([
    {$group:{_id:'$gender',counter:{$sum:1}}},
    {$project:{_id:0,counter:1}}
])
```

$match  
用于过滤数据，只输出符合条件的文档，兼容大多数find的查询表达式。  
```
# 查询name字段子文档中last字段的值为joe的文档
db.user.aggregate([
    {$match:{"name.last":"joe"}}
])
```

$group
```
# 按照年龄分组
db.user.aggregate([
    {$group:{_id: "$age"}}
])
```

#### 3.3 管道表达式

管道操作器的值即管道表达式，每个管道表达式是一个文档结构，由字段名、字段值和一些表达式操作符组成。管道表达式很多跟find中使用的表达式类似，如`$or`、`$sort`等。  

```
# 求和$sum
db.user.aggregate([{$group: {_id: "$name", nameSum: {$sum: "$age"}}}])

```

