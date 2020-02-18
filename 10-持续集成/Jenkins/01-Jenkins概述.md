## 一 Jekins初识


## 二 Jekins安装

下载Jenkins并安装（注意jekins需要java8以上环境）
```
wget https://pkg.jekins.io/redhat/jenkins-2.83-1.1.noarch.rpm
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

