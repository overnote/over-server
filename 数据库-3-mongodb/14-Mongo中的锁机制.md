## 一 Mongo中的锁

#### 1.1 Mongo中的锁简介

MongoDB允许多个客户端读写同一个数资源，为了保证数据的一致性，mongo提供了锁和并发控制等机制，防止同一部分数据同时被几个客户端修改。  

#### 1.2 锁粒度

3.4斑斑中，Mongo提供了多种锁粒度机制：
- Global 全局锁
- Database 库级锁
- Collection 集合锁
- Documend 文档锁，只有WiredTiger引擎支持
  
每个类型的锁分为read（读）和write（写）锁，其中read为共享锁（S），write为排他锁（X），意向锁（IS）和意向拍他所（IX）。  

意向表示read或者write操作想要获取更加细粒度的资源。  

