## 一 分布式session一致方案

由于用户访问的具体实例才会拥有session，这就造成同一个用户在多次访问时，因为访问到的服务实例不同，而造成session不一致。  

解决方案：
- session同步法：在服务之间同步session，每个实例都有session，该方式会占据大量的数据存储、网络资源
- nginx的ip_hash策略：该策略会让同一用户一直请求同一个实例。该方式让业务与session解耦，但是如果web服务重启，session就会消失，且如果web服务水平扩展后，根据用户IP重新Hash后，session会重新分布，导致一部分用户无法路由到正确的Session
- session集中管理：可以利用缓存服务如redis进行统一管理