## 一 Jekins初识

Jenkins是由Java开发的老牌持续集成工具，官网：https://jenkins.io/download    

Jenkins易于安装配置，具备完善的web管理界面，能够集成RSS/E-mail用来发布构建结果，生成测试报告，同时Jenkins支持分布式构建，也提供了大量的插件，如：git、svn、maven等。  






## 二 Jekins安装

### 2.1 传统安装方式

Jenkins可以由Java命令直接运行，或者通过web服务器（tomcat）运行，当然笔者推荐使用docker或者k8s以容器方式运行。  

下载Jenkins并安装（注意jekins需要java8以上环境）
```
wget https://pkg.jenkins.io/redhat/jenkins-2.83-1.1.noarch.rpm
rpm -ivh jenkins-2.83-1.1.noarch.rpm
```

配置Jekins，`vim /etc/sysconfig/jenkins`：
```
JENKINS_USER="root"
JENKINS_PORT="8888"
```

启动Jekins：
```
systemctl start jenkins
```

访问web管理界面：`localhost:8888`，进入洁面后选择左侧按钮，自动安装插件即可！  

### 2.2 docker安装

