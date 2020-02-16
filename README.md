## 一 开宗明义

### 1.0 开篇说明

本笔记特点：力争让读者无论从任何一个大章节阅读，都能知晓上下文，不会出现知识断层！！！！（当然也带来了缺点：会出现重复笔记）  

### 1.1 集群与分布式

很多时候，我们在讨论技术架构时，都离不开集群、分布式以及现在的微服务。为了厘清三者关系，请看下方示例：  

> 一个任务，1台机器执行1个任务需要1小时完成，执行10个任务就需要10个小时，现在购置10台机器后，如何提升效率？
- 集群方案：10台机器都部署该任务，若执行1个任务，只需要其中1台机器执行，时长1小时。若执行10个任务，每台执行1个，总耗费时间仍为1小时。
- 分布式方案：将该任务拆分成10个子任务，每个子任务只需要0.1小时完成，10台机器分别部署不同的子任务，1个任务完成总计需要0.1小时，10个任务完成耗费1小时。

**集群与分布式**：  
- 集群：代表业务的物理的形态，某个业务同时部署在了多个服务器上，通过提高单位时间内执行的任务数来提升效率。
- 分布式：代表业务的工作形态，某个业务被拆分成了多个子业务，子业务分别部署在不同的服务器上，通过缩短单个任务的执行时间来提升效率

由上我们可以看出，分布式中的每一个节点（服务器），都可以作为集群部署，而集群就不一定是分布式的了。   

好的设计应该是分布式和集群的集合，先分布式再集群，具体实现就是业务拆分成很多子业务，然后针对子业务进行集群部署，这样每个独立的子业务出现了问题，不会对整个系统造成影响。  

### 1.2 微服务与SOA

**微服务**：  
微服务只是一种架构风格，将一个大型软件拆分为多个松耦合的微服务，各个微服务可以独立部署，即将业务拆分为多个独立的单元，单元之间通过网络实现数据交互。  

**SOA**：  
业务系统分解为多个组件，让每个组件独立提供离散、自治、可复用的服务能力，通过服务的组合和编排来实现上层的业务流程。  

笔者这里认为微服务其实是分布式理念按照SOA的设想后的一种严格实现，但是微服务的应用可以部署在多台服务器上，也可以在同一个服务器（原则上不允许）。  

微服务相比分布式服务来说，服务粒度更小，服务之间耦合度更低，分布式服务最后都会向微服务架构演化，这是一种趋势，不过微服务化后也会带来很多新的挑战，如：服务粒度小、数量大造成的运维艰难，当然这时候也诞生了针对该问题的解决方案，如docker，k8s。    

### 1.3 分布式系统概念

分布式系统由一组计算节点组成，这些节点之间通过通过网络进行通信，并且能够协调工作以完成共同的任务，通过分布式，让多个廉价的计算机通过协作来达到昂贵的大型机的处理能力。相对而言，降低了成本。  

常见的分布式应用主要包括：
- 与分布式存储（storage）：每个节点存一部分数据，实践方式比如hadoop的hdfs
- 分布式计算（computation）：对计算任务进行切换，每个节点算一些，实践方式比如hadoop的mapreduce 

分布式理论最大的挑战是如何将任务分发到不同的计算机节点，可以利用分片机制（partition）。  

分布式系统特性与标准：
- 透明性：用户无需关心分布式系统如何实现，也不关心读到的数据来自哪个节点，在使用体验上，与单机系统无异
- 伸缩性：任务增加的时，分布式系统的处理能力需要随之增加，任务规模缩减时，多余机器可以裁撤
- 可用性：通过不可用时间与正常服务时间的比值来衡量
- 可靠性：计算结果正确、存储的数据不丢失
- 高性能：支持高并发与低延时，即单位时间内处理的多好，每个任务的处理时间短。
- 一致性：一致性有很多等级，一致性越强，对用户越友好，但会制约系统的可用性；一致性等级越低，用户就需要兼容数据不一致的情况，但系统的可用性、并发性很高很多。

### 1.4 分布式系统问题

分布式与单机系统相比，会遇到更多的挑战：
- 单个节点的故障（进程crash、断电、磁盘损坏）是小概率事件，但是节点越多，事故概率也就越高（指数级增加）
- 节点之间通过网络通信，网络本身可能出现断网、高延迟的情况

解决上述问题的办法是：冗余或者复制集（Replication），即多个节点负责同一个任务，最为常见的就是分布式存储中，多个节点复杂存储同一份数据，以此增强可用性与可靠性。同时，Replication也会带来性能的提升，比如数据的locality可以减少用户的等待时间。  

Partition和Replication是解决分布式系统问题利器，但也引入了更多的问题，最常见的问题是一致性问题：因为复制集各个副本之间需要保持数据的一致性，一致性在系统的角度和用户的角度又有不同的等级划分。如果要保证强一致性，那么会影响可用性与性能，在一些应用（比如电商、搜索）是难以接受的。如果是最终一致性，那么就需要处理数据冲突的情况，这也是不同的一致性解决方案理论的诞生缘由，如：CAP、FLP。

### 1.5 简单的分布式架构

架构图如下所示：  
![](./images/arch/readme01.png)  

在客户端，用户使用Web、APP、SDK，通过HTTP、TCP连接到分布式系统后：
- 第一个问题：选择哪个节点来提供服务？
    - **负载均衡**：通常使用负载均衡（load balance）解决
- 第二个问题：被负载到的服务器节点如何处理请求？
    - **分布式缓存**：简单的请求，比如读取数据，那么很可能是有缓存的，即分布式缓存，如果缓存没有命中，那么需要去数据库拉取数据。
    - **rpc**：对于复杂的请求，可能会调用到系统中其他的服务。假设服务A需要调用服务B的服务，首先两个节点需要通信，网络通信都是建立在TCP/IP协议的基础上，但是，每个应用都手写socket是一件冗杂、低效的事情，因此需要应用层的封装，因此有了HTTP、FTP等各种应用层协议。当系统愈加复杂，提供大量的http接口也是一件困难的事情。因此，有了更进一步的抽象，那就是RPC（remote produce call），使得远程调用就跟本地过程调用一样方便，屏蔽了网络通信等诸多细节，增加新的接口也更加方便。
    - **分布式事务**：一个请求可能包含诸多操作，即在服务A上做一些操作，然后在服务B上做另一些操作。比如简化版的网络购物，在订单服务上发货，在账户服务上扣款。这两个操作需要保证原子性，要么都成功，要么都不操作。分布式事务是从应用层面保证一致性：某种守恒关系。
    - **服务注册与发现**：上面说到一个请求包含多个操作，其实就是涉及到多个服务，分布式系统中有大量的服务，每个服务又是多个节点组成。那么一个服务怎么找到另一个服务（的某个节点呢）？通信是需要地址的，怎么获取这个地址，最简单的办法就是配置文件写死，或者写入到数据库，但这些方法在节点数据巨大、节点动态增删的时候都不大方便，这个时候就需要服务注册与发现：提供服务的节点向一个协调中心注册自己的地址，使用服务的节点去协调中心拉取地址。
- 第三个问题：如何处理日志？
    - **消息队列**：请求操作会产生一些数据、日志，通常为信息，其他一些系统可能会对这些消息感兴趣，比如个性化推荐、监控等，这里就抽象出了两个概念，消息的生产者与消费者。那么生产者怎么讲消息发送给消费者呢，RPC并不是一个很好的选择，因为RPC肯定得指定消息发给谁，但实际的情况是生产者并不清楚、也不关心谁会消费这个消息，这个时候消息队列就出马了。简单来说，生产者只用往消息队列里面发就行了，队列会将消息按主题（topic）分发给关注这个主题的消费者。消息队列起到了异步处理、应用解耦的作用。
    - **分布式运算**：用户操作会产生一些数据，这些数据忠实记录了用户的操作习惯、喜好，是各行各业最宝贵的财富。比如各种推荐、广告投放、自动识别。这就催生了分布式计算平台，比如Hadoop，Storm等，用来处理这些海量的数据。
- 第四个问题：用户数据如何持久化？
    - **分布式存储**：用户的数据需要持久化，但数据量很大，大到按个节点无法存储，那么这个时候就需要分布式存储：将数据进行划分放在不同的节点上，同时，为了防止数据的丢失，每一份数据会保存多分。传统的关系型数据库是单点存储，为了在应用层透明的情况下分库分表，会引用额外的代理层。

### 1.6 分布式常见的技术实现

```
架构体系：  SOA、微服务、云原生
负载均衡：  Nginx、LVS
开发框架：  SpringCloud、ServiceComb、go-micro、go-kit
容器领域：  Docker、Kubernetes、Prometheus、Istio
缓存系统：  memcache、redis
协调中心：  zookeeper、etcd、consul
rpc框架：   grpc、dubbo、brpc
消息队列：  kafka、rabbitMQ、rocketMQ
实时数据平台：  storm、akka
离线数据平台：  hadoop、spark
数据存储：  mysql、oracle、MongoDB、HBase
数据搜索：  elasticsearch、solr
日志系统：  rsyslog、elk、flume
```

## 二 架构相关书籍

### 2.1 架构基础

- [从零开始学架构](https://book.douban.com/subject/30335935/)：简单认识架构，并实践一些常见互联网技术
- [互联网创业核心技术:构建可伸缩的web应用](https://book.douban.com/subject/26906846/)：认识当前互联网架构的一些常见技术

### 2.2 分布式理论

经典书籍：
- [数据密集型应用系统设计](https://book.douban.com/subject/30329536/)：即ddia
- [大数据日知录](https://book.douban.com/subject/25984046/)
- [nosql精粹](https://book.douban.com/subject/25662138/)  

下列书籍评分不高，但是部分章节值得一看：
- [从Paxos到Zookeeper分布式一致性原理与实践](https://book.douban.com/subject/26292004/)
- [深入分布式缓存：从原理到实践](https://book.douban.com/subject/27602483/)
- [etcd技术内幕](https://book.douban.com/subject/30275551/)
- [分布式实时处理系统：原理、架构与实现](https://book.douban.com/subject/26833829/)：利用C++开发一套分布式实时处理系统
- [大规模分布式存储系统](https://book.douban.com/subject/25723658/)：构建分布式存储系统必备


### 2.3 微服务

微服务设计：
- [微服务架构设计模式](https://book.douban.com/subject/33425123/)：
- [微服务设计](https://book.douban.com/subject/26772677/) ：学习微服务的一些基础概念，本书可以最后看
- [微服务设计原理与架构 郑天民](https://book.douban.com/subject/30233793/)：冷门书籍，但是讲解也不错
- [未来架构-从服务化到云原生](https://book.douban.com/subject/30477839/)：适合大致领略下云与安生当下的一些技术体系
- [大型企业微服务架构实践与运营](https://book.douban.com/subject/30449480/)：对微服务技术、理念的总结

微服务落地：
- [架构探险：轻量级微服务架构（下册](https://book.douban.com/subject/27115266/)：下册的微服务基本实践不错
- [架构修炼之道](https://book.douban.com/subject/33389549/)
- [持续演进的Cloud Native](https://book.douban.com/subject/30370644/)
- [亿级流量网站架构核心技术](https://book.douban.com/subject/26999243/)
- [微服务架构实战 郑天民](https://book.douban.com/subject/30417709/)

### 2.4 架构之术

- [代码整洁之道](https://book.douban.com/subject/4199741/)
- [架构整洁之道](https://book.douban.com/subject/30333919/) 
- [重构:改善既有代码的设计 第2版](https://book.douban.com/subject/30468597//)
- [架构即未来](https://book.douban.com/subject/26765979/)
- [系统架构:复杂系统的产品设计与开发](https://book.douban.com/subject/26938710/)
- [企业IT架构转型之道](https://book.douban.com/subject/27039508/)
- [银行信息系统架构](https://book.douban.com/subject/26677445/)
- [面向模式的软件架构](https://book.douban.com/subject/4848563/)
- [设计模式](https://book.douban.com/subject/1052241/)


## 三 数据库书籍

通用类：
- [数据库系统概念](https://book.douban.com/subject/10548379/)
- [数据库系统实现](https://book.douban.com/subject/4838430/)
- [数据库索引设计与优化](https://book.douban.com/subject/26419771/)
- [数据库查询优化器的艺术](https://book.douban.com/subject/25815707/)
- [SQL基础教程](https://book.douban.com/subject/27055712/)
- [分布式数据库系统原理](https://book.douban.com/subject/26851605/)

MySQL：
- [涂抹MySQL](https://book.douban.com/subject/25898562/)：基础入门
- [高性能MySQL](https://book.douban.com/subject/23008813/)：mysql开发技巧提升
- [MySQL性能调优与架构设计](https://book.douban.com/subject/3729677/)：MySQLDBA宽泛学习
- [MySQL技术内幕 InnoDB存储引擎 第2版](https://book.douban.com/subject/24708143/)：MySQLDBA深入学习，豆瓣未收录，仍然使用第1版

Redis：
- [Redis开发与运维](https://book.douban.com/subject/26971561/)：基础入门
- [Redis深度历险](https://book.douban.com/subject/30386804/)：开发技巧提升
- [Redis设计与实现](https://book.douban.com/subject/25900156/)：redis源码与架构

MongoDB：
- [MongoDB权威指南](https://book.douban.com/subject/25798102/)
- [MongoDB实战](https://book.douban.com/subject/19977785/)

PostgreSQL：
- [PostgreSQL技术内幕](https://book.douban.com/subject/30256561/)
- [PostgreSQL 数据库内核分析](https://book.douban.com/subject/6971366/)

Oracle：
- [涂抹Oracle](https://book.douban.com/subject/4196676/)
- [收获，不止Oracle](https://book.douban.com/subject/23857303/)
- [Oracle Database 9i/10g/11g编程艺术](https://book.douban.com/subject/5402711/)
- [Oracle性能诊断艺术](https://book.douban.com/subject/4076215/)
- [Oracle高效设计](https://book.douban.com/subject/1503909/)
- [构建Oracle高可用环境](https://book.douban.com/subject/2531036/)
- [DBA的思想天空](https://book.douban.com/subject/19966085/)
- [Oracle RAC日记](https://book.douban.com/subject/4838427/)
- [深入解析Oracle](https://book.douban.com/subject/3393767/)
- [Oracle DBA手记2](https://book.douban.com/subject/5362865/)
- [海量数据库解决方案](https://book.douban.com/subject/5346169/)

## 四 其他常用软件

### 4.1 web服务器
- [深入理解Nginx：模块开发与架构解析 第2版](https://book.douban.com/subject/26745255/)
- [Tomcat架构解析](https://book.douban.com/subject/27034717/)
- [深入剖析Tomcat](https://book.douban.com/subject/10426640/)

### 4.2 消息队列
- [RabbitMQ实战指南](https://book.douban.com/subject/27591386/)
- [Kafka权威指南](https://book.douban.com/subject/27665114/)
- [Apache Kafka源码剖析](https://book.douban.com/subject/27038473/)

### 4.3 Docker&k8s

docker:
- [深入浅出Docker](https://book.douban.com/subject/30486354/)
- [Docker容器与容器云 第2版](https://book.douban.com/subject/26894736/)

k8s:
- [Kubernetes in Action](https://book.douban.com/subject/30418855/)
- [Service Mesh微服务架构设计](https://book.douban.com/subject/34856113)
- [云原生服务网格Istio：原理、实践、架构与源码解析](https://book.douban.com/subject/34438220/)
- [Prometheus监控实战](https://book.douban.com/subject/34801408/)

## 附录：笔记汇总

**OverNote**地址：https://github.com/overnote    
**笔者的地址**：https://github.com/ruyuejun  

**OverNote分类**：  
- [Golang](https://github.com/overnote/over-golang)：详尽的Go领域笔记：Go语法、Go并发编程、GoWeb编程、Go微服务等
- [大前端](https://github.com/overnote/over-javascript)：包含JavaScript、Node.js、vue/react、微信开发、Flutter等大前端技术
- [数据结构与算法](https://github.com/overnote/over-algorithm)：以Go实现为主记录数据结构与算法的笔记，附带C、JS版本
- [服务端架构](https://github.com/overnote/over-server)：分布式与微服务笔记，附Nginx、Mysql、Redis等常用服务端技术
- [Linux](https://github.com/overnote/over-linux)：计算机组成原理、操作系统、计算机网络基础学科笔记，完善中
- [大数据](https://github.com/overnote/over-bigdata)：大数据笔记，完善中
- [Python与机器学习](https://github.com/overnote/over-python)：Python相关笔记，完善中  