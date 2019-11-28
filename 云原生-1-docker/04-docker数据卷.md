## 一 docker数据管理

在docker中，数据的持久化、多容器之间数据的共享，使用的是数据卷(Data Volumes)和数据卷容器(Data Volume Containers)。  

数据卷(Data Volumes)：容器内数据直接映射到本地主机环境。
- 数据卷可以在容器之间共享和重用，本地与容器间传递数据更高效
- 对数据卷的修改会立马有效，容器内部与本地目录均可
- 对数据卷的更新，不会影响镜像，对数据与应用进行了解耦操作
- 卷会一直存在，直到没有容器使用

数据卷相关的命令：在docker run 时添加 -v 参数（即--volume），挂载一个或者多个数据卷到当前容器，默认为空
```
# 命令格式：docker run -itd --name [容器名字] -v [宿主机目录]:[容器目录][镜像名称] [命令(可选)]       
docker run -itd --name test1 -v /home/hello/tmp/:/test1/ nginx
```

贴士：也可以挂载文件，但是极度不推荐，因为文件可以进行编辑，导致改变，这会引起docker错误。  

## 二 数据卷容器

#### 2.1 数据卷容器介绍

数据卷容器（Data Volume Containers）是专门用来维护数据卷的容器。需要在多个容器之间共享一些持续更新的数据，最简单的方式是使用数据卷容器。  

数据卷容器也是一个容器，专门用来提供数据卷供其他容器挂载。  

贴士：数据卷容器自身并不需要启动，但是启动的时候依然可以进行数据卷容器的工作。  

#### 2.2 数据卷容器操作

创建一个数据卷容器：
```
# 格式 docker create -v [容器数据卷目录] --name [容器名字][镜像名称] [命令(可选)] 
docker create -v /data --name v1-test1 nginx

# 创建1个新容器，并挂载数据容器卷
docker run --volumes-from 4693558c49e8 -tid --name vc-test1 nginx /bin/bash
```

#### 2.3 数据备份

步骤：
- 1、创建一个挂载数据卷容器的容器
- 2、挂载宿主机本地目录作为备份数据卷
- 3、将数据卷容器的内容备份到宿主机本地目录挂载的数据卷中

示例：
```
# 创建备份目录:
mkdir /backup/

# 创建备份的容器:
docker run --rm --volumes-from 60205766d61a -v /home/itcast/backup/:/backup/ nginx tar zcPf /backup/data.tar.gz /data

# 验证操作:
ls /backup
zcat /backup/data.tar.gz
```

#### 2.4 数据恢复

步骤：  
- 1、创建一个新的数据卷容器(或删除原数据卷容器的内容) 
- 2、创建一个新容器，挂载数据卷容器，同时挂载本地的备份目录作为数据卷 
- 3、将要恢复的数据解压到容器中

```
# 启动数据卷容器
docker start c408f4f14786

# 删除源容器内容
$ docker exec -it vc-test1 bash root@c408f4f14786:/# rm -rf /data/*

# 恢复数据
docker run --rm --volumes-from v-test -v /home/itcast/backup/:/backup/ nginx tar xPf /backup/data.tar.gz -C /data

# 验证
docker exec -it vc-test1/bin/bash root@c408f4f14786:/# ls /data/data/ v-test1.txt v-test2.txt

# 新建新的数据卷容器
docker create -v /newdata --name v-test2 nginx

# 建立新的容器挂载数据卷容器
docker run --volumes-from a7e9a33f3acb -tid --name vc-test3 nginx /bin/bash 

# 恢复数据
docker run --rm --volumes-from v-test2 -v /home/test/backup/:/backup/ nginx tar xPf /backup/data.tar.gz -C /newdata

# 验证
docker exec -it vc-test3 /bin/bash
root@c408f4f14786:/# ls /newdata
v-test1.txt v-test2.txt
```

注意: 解压的时候，如果使用目录的话，一定要在解压的时候使用 -C 制定挂载的数据卷容器，不然的话容器数据 是无法恢复的，因为容器中默认的backup目录不是数据卷，即使解压后，也看不到文件。  