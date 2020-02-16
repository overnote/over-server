## 一 数据卷

在docker中，数据的持久化、多容器之间数据的共享使用的是数据卷(Data Volumes)和数据卷容器(Data Volume Containers)。  

数据卷(Data Volumes)：容器内数据直接映射到本地主机环境，即文件挂载，命令如下：
```
# 命令格式
docker run -it --name [容器名字] -v [宿主机目录]:[容器目录][镜像名称] [命令(可选)]  

# 命令示例
docker run -it --name=mycontainer1 -v /home/data:/data nginx
```

贴士：
- 一个容器可以挂载多个目录
- 文件也可以拷贝到数据卷：docker cp 宿主机内文件 容器名:容器目录 
- 该命令也可以用于从容器内拷贝出文件到本地，将cp后两个命令倒置即可

注意：虽然挂载文件可以实现容器与宿主机之间的通信，但是由于件可以进行编辑，导致改变，这会引起docker错误。

## 二 数据卷容器

### 2.1 数据卷容器介绍

数据卷容器（Data Volume Containers）是专门用来维护数据卷的容器，需要在多个容器之间共享一些持续更新的数据，最简单的方式是使用数据卷容器。  

数据卷容器也是一个容器，专门用来提供数据卷供其他容器挂载。  

贴士：数据卷容器自身并不需要启动，但是启动的时候依然可以进行数据卷容器的工作。  

### 2.2 数据卷容器操作

创建(也可以直接启动)一个数据卷容器：
```
docker run -it --name=c0 -v /volume centos:7
```

其他容器使用该数据卷
```
docker run -it --name=c1 --volumes-from c0 centos:7
docker run -it --name=c2 --volumes-from c0 centos:7
```

## 三 容器的数据备份与恢复

### 3.1 数据备份

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

### 3.2 数据恢复

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