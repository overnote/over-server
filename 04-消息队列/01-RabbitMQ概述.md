## 安装

mac安装方式：
```
# 替换homebrew源
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
source ~/.bash_profile

# 安装rabbitmq
brew update
brew install rabbitmq
echo 'export PATH=$PATH:/usr/local/opt/rabbitmq/sbin' >> ~/.bash_profile
source ~/.bash_profile

# 安装管理界面
rabbitmq-plugins enable rabbitmq_management

# 启动
rabbitmq-server

# 停止
rabbitmqctl stop
```

启动：
```
Linux启动方式：
systemctl start rabbitmq-server     // 启动服务
rabbitmqctl stop                    // 停止
```

常用命令：
```
rabbitmq-plugins list               # 列出所有插件，启用的插件会显示 [e*]
rabbitmq-plugins enable 插件名      # 启用插件，disable卸载 如常用的管理界面 rabbitmq_management，
```

## 管理界面

管理界面地址：http://localhost:15672/  

默认登录账户和密码都是：guest

## 入门指引

进入管理界面后，点击Admin菜单：
- 首先进入界面右侧的Virtual Hosts添加一个 Virtual Host
- 然后点击界面右侧的Users，添加一个用户，注意添加用户时必须设置用户的mod
- 在用户列表点击新建的用户名为该用户添加刚才的 Virtual Host 权限

虚拟主机名是为了保证该用户只在该域下执行操作，做到环境隔离，比如在同一台服务器上，可以利用Virtual Host 来区分开发环境和生产环境。 

## 五个模式

### Simple 简单模式


### Work 工作模式

当生产速度大于消费速度时，推荐使用该模式，以实现负载。  

其代码与Simple模式一致，只是多开启了消费端，且一个消息只能被一个消费者获取。

### Publish/Subscribe 订阅模式

消息被路由投递给了多个队列，订阅了该队列的多个消费者都能够获取。  

此时需要配置exchange交换机。  

### Routing 路由模式

一个消息可被多个消费者获取，且消息的目标队列可被生产者指定  

