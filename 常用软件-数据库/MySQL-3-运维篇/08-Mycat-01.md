## 一 Mycat概述

Mycat是开源的数据库集群中间件，是一款可以用来替代MySQL的加强版数据库，支持事务，同时也融合了内存缓存技术、Nosql技术、HDFS等。  

## 二 Mycat集群搭建

### 2.0 服务器准备

MySQL集群1：  

| 角色 | 主机ip | 端口 | 主机hostname |  
| master | 192.168.1.18 | 3306 | master01 |  
| slave  | 192.168.1.18 | 3307 | slave01  |  


MySQL集群2：  

| 角色 | 主机ip | 端口 | 主机hostname |  
| master | 192.168.1.18 | 3316 | master02 |  
| slave  | 192.168.1.18 | 3317 | slave02  |  

### 2.1 配置master
