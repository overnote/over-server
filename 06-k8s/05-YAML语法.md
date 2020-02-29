## 一 什么是资源清单文件

使用kubectl命令固然可以在k8s及群众进行简单的操作，但是k8s集群一般会管理着成千上万的资源，对这些资源对象的编排部署仅仅依靠命令是十分低效的。  

K8S集群中对资源管理、资源对象的编排可以通过声明样式(YAML)文件来解决，即一般使用YAML文件创建pod，这样的YAML文件即资源清单文件这种文件即资源清单文件。   

## 二 YAML文件书写格式

### 2.1 YAML基本格式

YAML是一种标记语言，该语言以数据为中心，而不强调标记，其可读性很高，可以方便的用来表达数据序列！  

YAML的基本语法：
- 空格作为缩进，键值之间必须有空格
- 缩进的空格数不重要，只要相同层级的元素左侧对其即可
- 低版缩进时不允许使用Tab
- #是注释，会将一整行全部注释

YAML支持三种数据结构：对象、数组、纯量（scalars）

### 2.2 YAML中的对象

对象一般是键值对的集合，在YAML中表示如下：
```yaml
name: Tom
age: 18
```

也可以采用行内表示：
```yaml
hash: { name: Tom, age: 18}
```

### 2.3 YAML中的数组

数组的基本表示方法：
```yaml
People
- Tom
- Jack
```

也可以采用行内表示：
```yaml
People: [Tom, Jack]
```

### 2.4 YAML中的纯量

纯量是指单个的、不可再分的值：
```yaml
# 数值直接以字面量的形势表示
number: 13.1
# 布尔值只能用true、false
isSet: true
# null 用 ~ 表示
parent: ~
# 时间采用 ISO8601 格式
iso8601: 2001-12-14t21:59:43.10-05:00
# 日期采用复合格式
date: 2001-12-14
# !!两个感叹号表示强制转换类型
e: !!str 123
f: !!str true
```

### 2.5 YAML中的字符串

YAML中的字符串默认不需要用引号包含，但是如果字符串中包含空格、特殊字符，则需要引号：
```yaml
str: '内容：   字符串'
```

单引号之中如果还有单引号，必须连续使用两个单引号进行转义：
```yaml
str: 'labor''s day'
```

字符串可以写成多行，但是从第二行开始，必须有一个单空格进行缩进，换行符会被转换为空格
```yaml
str: 这是一段
  多行
  字符串
```

多行字符串可以使用 | 保留换行符，也可以使用 > 折叠换行
```yaml
this: |
Foo
Bar
that
Foo
Bar
```

## 三 资源清单描述

贴士：yaml中起的名字如果需要分割，最好是"-"分割。   

常用字段：
```yaml
# 这里是k8s api的版本，目前是v1，kubectl api-versions 可查看
apiVersion: v1
# yam文件定义的资源类型和角色，如Pod
kind: Deployment
# 元数据对象
metadata:
  name:  <deploy-name>      # 元数据对象名字，如Pod的名字
  namespace: <ns-name>      # 元数据对象的命名空间
  labels:                   # 元数据对象的标签
    <key>: <value>
# 资源对象的详细值
spec: 
  replicas: 2   
  selector:
    matchLabels: 
      <key>: <value>
  revisionHistoryLimit: 5
  progressDeadlineSeconds: 600  #可选参数；deployment 卡住执行出现错误时，等待deployment 进行的时间秒数，deployment controller会继续重试。设置该值必须大于.spec.minReadySeconds。
  minReadySeconds: 0  #可选参数，默认为0（pod 在ready后就被认为可用），pod中容器没有crash 并被认为可用状态的最小秒数。
  strategy:   #更新的策略，默认为rollingUpdate , 可选Recreate 在创建新pod之前会杀掉所有已经存在的pod
    rollingUpdate:
      maxUnavailable: 25%   #最大不可用比例，可绝对值，可比例
      maxSurge: 25%     #最大更新pod量，可绝对值，可比例，按上取整
  rollbackTo:  #可选参数，用来配置deploy回退的配置，设置该参数将触发回退操作，每次回退完成会清楚该值
    revision: 0  #默认为0，用来指定回退到的revision，0意味着回退到历史中最老的revision
  paused: false  #可选参数 默认非paused,boolean值。用来指定暂停和恢复deploy,paused 和 非paused的唯一区别在于，paused后，所有对PodTemplateSpec的修改都不会触发新的rollout
  template: #和pod template中一样
    metadata:
      name: <pod-name>
      labels:
        <key>: <value> #需要和sepc.selector.matchLabels 中的lable 匹配，否则rs 服务管理到该pod
    spec:
      containers:
      - name: 
        image:
        imagePullPolicy: Alway # Alway,IfNotPresent,Never  
        command:
        args:
        workingDir: #指定容器的工作目录
        resources: #容器运行的资源申请和限制
          requests: #申请
            cpu:
            memory:
          limits: #最大限制
            cpu:
            memory:
        ports: #容器应用运行暴露的port
        - name:
          containerPort:
          protocol:
        env: # 指定容器运行的环境变量
        - name:
          value:
        volumeMounts: # 挂载外部文件到容器
        - name: <volume-name> # volume 的 name
          mountPath: /home # 挂载到容器中的指定路径
        livenessProbe:  #存活探针
          httpGet: #http检测形式
            path: </path>  #路径
            port: <containerPort>  #端口
            httpHeaders:  #需要设置请求头时的设置
            - name: <key>
              value: <value>
          initialDelaySeconds: 20 #开始探针检测的等待秒数，根据业务需求具体启动时间来设置，过长会造成应用长时间无法提供服务，过短会杀死掉没有启动完全的应用
          #exec:  探针的exec形式
          #  command:
          #  - cmd 指令
          #  - args 参数
          periodSeconds: 10 # 执行探测的间隔
          timeoutSeconds: 3  # 探测超时的时间
          successThreshold: 1 # 默认为1，失败状态后的服务检测成功1次后就识别为成功
          failureThreshold: 3  # 成功状态后的服务，检测3次失败后为失败
        readinessProbe: # 就绪探针
          tcpSocket: #tcp 端口探测模式，kubelet 执行检测工作
            port: <containerPort> #需要探测容器端口
        terminationMessagePath: /dev/terminatino-log
        terminationMessagePolicy: File
        securityContext: {} #容器上下文权限
        lifecycle: # 生命周期
          postStart:
            exec:
              command:
          preStop:
            exec:
              command:
        stdin: true #标准输入,是否开启
        stdinOnce: true #stdin 为true 后，可以有打开多个通道连接容器，当stdinOnce为true,只能有一个通道连接，并且关闭通道后，stdin也将关闭，直到容器重启
        tty: true #是否开启交互窗口，加stdin 是docker -it 一样的功能
      hostAliases: # 追加pod 中/etc/hosts 文件内容
      - ip: "10.1.2.3"
        hostnames:
        - "foo.remote"
        - "bar.remote"
      shareProcessNamespace: true  # 几乎不用参数，pod 中所有容器共享进程空间
      nodeSelector: #指定调度节点标签，用节点label 匹配
        <key>: <value> 
      dnsPolicy: ClusterFirst  # dns策略
      restartPolicy: Always #Always,Never,Failure 重启策略，总是、永不、失败后重启
      schedulerName: default-scheduler  # 默认调度策略
      securityContext: {} #pod上下文权限
      terminatinoGracePeriodSeconds: 30  # 默认30 ，优雅关闭时间
```

