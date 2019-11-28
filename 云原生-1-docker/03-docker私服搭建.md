docker私有仓库（用于局域网内共享一个镜像）：

步骤：
```
# 拉取私有镜像
docker pull registry  
# 启动私有仓库容器
docker run -di --name=registry -p 5000:5000 registry
# 访问ip查看：localhost:5000/v2/_catalog     显示一个 repositories:[]  则创建成功    
# 修改配置，让docker信任本机的私有仓库地址
vim /etc/docker/daemon.json

{
    "registry-mirrors": ["http://f1361db2.m.daocloud.io"],
    "insecure-registries":["192.168.86.131:500"]
}
# 重启docke软服务
systemctl restart docker
# 再次启动docker 私服
docker start registry
# 给要上传的镜像打个标签
docker tag jdk1.8 192.168.86.131:5000/jdk1.8
# 上传到私服
docker push 192.168.86.131:5000/jdk1.8
```