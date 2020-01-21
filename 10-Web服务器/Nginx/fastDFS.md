## 一 fastDFS简介

fastDFS是用C语言编写的一款开源分布式文件系统。fastDFS专门为互联网量身定制，注重高可用、高性能等指
标：
- 冗余备份：纵向扩容
- 线性扩容：横向扩容, 增加容量

利用fastDFS可以很容易搭建一套高性能的文件服务器集群提供文件 上传、下载 等服务。  

## 二 fastDFS架构

fastDFS中的三个角色：
- 追踪器 ( tracker ) ：管理者守护进程，第一个启动
- 存储节点 ( storage ) ：存储守护进程，可以理解为网络环境中可以存储文件的主机，第二个启动，用于存储文件
- 客户端（client）：开发者用于上传、下载的程序，最后启动


使用fastDFS上传过程：
如图：  ![](../images/hot/fastDFS-01.png)    
- 启动追踪器
- 启动存储节点
  - 主动连接追踪器，汇报当前存储节点的状态信息
  - 以后会定时汇报状态
- 客户端启动：发起一个上传请求
  - 连接追踪器，询问后得到哪个存储节点有足够的容量
  - 追踪查询存储节点信息
  - 将查到的节点信息发送给客户端
- 客户端通过得到的处处节点地址，连接存储节点
- 将文件上传到存储节点上，存储节点得到一个file_id，并将其发送给客户端
- 客户端需要存储这个fileID，用于下载


使用fastDFS下载过程：  
如图：  ![](../images/hot/fastDFS-02.png)  
- 先启动追踪器
- 启动存储节点
  - 主动连接追踪器, 汇报当前存储节点的状态信息
  - 后边定时汇报状态
- 客户端程序启动, 连接追踪器, 发送下载请求
  - 客户端询问追踪, 看那个存储节点上有要下载的文件
  - 追踪查询存储节点信息
  - 将查到的节点地址发送给客户端
- 客户端通过得到的存储节点地址, 连接存储节点
- 将存储节点发送给客户端的文件, 保存到本地

## 三 fastDFS集群

fastDFS的tracker集群可以有效避免单点故障，集群以轮询方式进行工作。  

fastDFS的存储节点集群以组的方式进行管理：
- 横向扩容 -> 添加新的分组, fastDFS容量增加了
  - 不同组的主机之间不通信
  - 各组的容量相加就是整体容量
- 纵向扩容 -> 在现有的组中添加新的主机, 和同组主机之间互为备份关系
  - 同组主机中存储的内容相同
  - 同组主机之间是通信的
  - 当前组的容量按照最小的主机来算

![](../images/hot/fastDFS-03.png)  

## 四 fastDFS安装

#### 4.1 安装

- libfastcommon-1.36.zip
  - fastDFS的基础库包
  - unzip libfastcommon-1.36.zip
  - cd libfastcommon-master
  - ./make.sh
  - sudo ./make.sh install
- fastdfs-5.10.tar.gz
  - tar zxvf fastdfs-5.10.tar.gz
  - cd fastdfs-5.10
  - ./make.sh
  - sudo ./make.sh install

#### 4.2 配置

fastDFS的配置文件默认存储目录: /etc/fdfs
```
client.conf.sample storage.conf.sample storage_ids.conf.sample tracker.conf.sample
```

tracker：
```
bind_addr=
    - 追踪器对应的主机的IP地址
    - 如果不写, 会自动绑定本机IP地址
    - 如果阿里云, 这个地方空着就行了, 否则有可能会无法启动
port=22122
    - 追踪器绑定的端口
    - 只要是一个空闲的没有被占用的端口就可以
base_path=/home/ruyue/fastdfs
    - 追踪器存储log日志或者一些进程文件相关的目录
    - 对应的路径必须要存在
```

storage:
```
group_name=group1
    - 当前存储节点所属的组
    - 横向扩容还是纵向扩容, 是通过该属性控制的
bind_addr=
    - 存储节点的IP地址
    - 如果不写, 会自动绑定本机IP地址
    - 如果阿里云, 这个地方空着就行了, 否则有可能会无法启动
port=23000
    - 客户端连接存储节点是时候使用的
    - 只要是一个空闲的没有被占用的端口就可以
base_path=/home/ruyue/fastdfs
    - 存储节点存储log日志的目录
    - 这个目录必须存在
store_path_count=2
    - 存储节点上, 存储文件的路径个数
    - 一块硬盘对应一个存储路径就可以
store_path0=/home/ruyue/fastdfs
store_path1=/home/ruyue/fastdfs1
    - 存储文件的具体目录
tracker_server=192.168.247.131:22122
    - 连接的追踪器的地址
tracker_server=192.168.247.132:22122
    - 追踪去集群的声明方式
```

client:
```
base_path=/home/ruyue/fastdfs
- 客户端写log日志的目录
tracker_server=192.168.0.197:22122
-客户端要连接的追踪器的地址
```

## 五 fastDFS使用

tracker：
```
# 启动
$ fdfs_trackerd /etc/fdfs/tracker.conf
# 停止
$ fdfs_trackerd /etc/fdfs/tracker.conf stop
# 重启
$ fdfs_trackerd /etc/fdfs/tracker.conf restart
```

storage:
```
# 启动
$ fdfs_storaged /etc/fdfs/storage.conf
# 停止
$ fdfs_storaged /etc/fdfs/storage.conf stop
# 重启
$ fdfs_storaged /etc/fdfs/storage.conf restart
```

client:
```
# 上传
$ fdfs_upload_file /etc/fdfs/client.conf 要上传的文件
# 下载
$ fdfs_download_file /etc/fdfs/client.conf FileID
$ fdfs_upload_file /etc/fdfs/client.conf a.yaml
group1/M00/00/00/wKj3g1v0zTuAW1zaAAAUrymD-Z449.yaml
    - group1 -> 文件上传到了哪个组
    - M00 -> store_path0
    - M01 -> store_path1
```

## 六 nginx与fastDFS整合

#### 6.1 安装

1. 在存储节点上安装Nginx, 将软件安装包拷贝到fastDFS存储节点对应的主机上
```
# nginx安装了fastDFs插件, 向让nginx和fastDFS存储节点进行数据通信
# 在一台主机上 同时 安装nginx 和 fastDFS{存储节点的角色}
```

2. 在存储节点对应的主机上安装Nginx, 作为web服务器
```
# fastDFS插件源码包 -> fastdfs-nginx-module_v1.16.tar.gz
$ tar zxvf fastdfs-nginx-module_v1.16.tar.gz
$ cd fastdfs-nginx-module -> 里边有安装源码
# 进入nginx的源码安装目录
$ tree -L 1

$ ./configure --add-module=fastdfs插件的源码根目录/src
$ make
$ sudo make install
```
#### 6.2 一些错误

make错误：
```
# 1. fatal error: fdfs_define.h: 没有那个文件或目录
# 2. fatal error: common_define.h: 没有那个文件或目录
# 需要修改 objs/Makefile文件
正确的头文件路径需要通过find 进行搜索
$ find / -name fdfs_define.h
```

![](../images/hot/fastDFS-04.png)    

Nginx没有worker进程：
```
$ sudo nginx
nginx: error while loading shared libraries: libpcre.so.1: cannot open shared object
file: No such file or directory
# 解决方案
$ sudo find / -name libpcre.so
/usr/local/lib/libpcre.so
$ sudo vi /etc/ld.so.conf
在这个文件中加一句话: /usr/local/lib/
$ sudo ldconfig
$ ps aux|grep nginx
root 85456 0.0 0.0 32960 444 ? Ss 17:16 0:00 nginx: master
process nginx
ruyue 85478 0.0 0.0 21536 1060 pts/1 S+ 17:18 0:00 grep --color=auto
nginx
# 发现问题, 没有worker进程
# 去看 logs/error.log
```

#### 6.3 解决步骤

1. 拷贝文件 mod_fdfs.conf
```
# 进入fastDFS插件源码安装目录
ruyue@ubuntu:src$ pwd
/home/ruyue/package/nginx/fastdfs-nginx-module/src
ruyue@ubuntu:src$ tree


$ sudo cp ./mod_fastdfs.conf /etc/fdfs
```

2.修改配置文件 mod_fdfs.conf - > 参考存储节点的配置文件进行修改
```
base_path=/home/ruyue/myfastDFS/storage
- 存储节点写log日志的目录
tracker_server=192.168.247.131:22122
- 存储节点连接追踪器的地址
storage_server_port=23000
- 存储节点绑定的端口
group_name=group1
- 当前存储节点所属的组
url_have_group_name = true
- 客户端访问fastdfs上存储的文件的地址的时候, url中是否包含组的名字
store_path_count=1
- 存储节点上存储路径的个数
store_path0=/home/ruyue/myfastDFS/storage
- 具体的存储目录
```

3.拷贝http.conf
```
# 从fastdfs源码安装目录中找
conf/http.conf
sudo cp http.conf /etc/fdfs
```

4. 拷贝mime.types
```
# 从nginx的源码安装目录中找
conf/mime.types
sudo cp mime.types /etc/fdfs
```

5. 访问：http://192.168.247.131
```
url: http://192.168.247.131/group1/M00/00/00/test.jpg
在nginx端要处理一个指令
/group1/M00/00/00
location /group1/M00/00/00
{ 

} 

M00 -> store_path0/data
location /group1/M00/
{
# fastDFS存储文件的目录
root /home/ruyue/myfastDFS/storage/data;
ngx_fastdfs_module; # 添加完成之后就可以和fastdfs存储节点通信
}
```