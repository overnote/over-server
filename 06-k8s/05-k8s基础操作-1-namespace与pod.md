## 一 命名空间 namespace

### 1.1 命名空间的介绍

命名空间用于在多租户的情况下，实现资源隔离，但是该隔离只是逻辑上的隔离，我们也可以针对每个namespace进行资源配额。  

查看所有命名空间：
```
# 执行(namespace 可以简写为ns)
kubectl get namespace

# 输出四个命名空间
NAME              STATUS   AGE
default           Active   3h44m
kube-node-lease   Active   3h44m
kube-public       Active   3h44m
kube-system       Active   3h44m
```

四个命名空间的说明：
- default：用户创建的pod默认位于该命名空间
- kube-public：所有用户都能访问
- kube-node-lease：集群节点的租约状态，k8sv1.13加入的空能
- kube-system：集群使用的命名空间，比如`kubectl get pods --namespace kube-system`

### 1.2 命名空间的使用

使用命令可以直接操作命名空间：
```
# 创建名为 myspace 的命名空间
kubectl create namespace myspace

# 删除名为 myspace 的命名空间
kubectl delete namespace myspace
```

当然资源清单也可以操作命名空间，在文件 nsdemo1.yaml中：
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: myspace
```

运行该资源清单：
```
# 执行创建
kubectl apply -f nsdemo1.yaml 

# 执行删除
kubectl delete -f nsdemo1.yaml 
```

## 二 Pod

### 2.1 Pod简介

Pod是k8s资源调度的最小单元，注意：k8s的最小单元并不是容器，一个Pod中包含了多个容器，笔者认为这样做能更好的做到资源分配、负载均衡。  

Pod相关的命令：
```
# 该命令默认查看default命名空间中的Pod，注意 pods可以简写为pod
kubectl get pods      # 指定查看Pod： kubectl get pods --n kube-system

# 显示结果
NAME                     READY   STATUS    RESTARTS   AGE
nginx-554b9c67f9-k99dn   1/1     Running   0          71m
```

### 2.2 使用资源清单创建Pod

编写一份创建含有nginx的Pod，文件名为 poddemo1.yaml。在书写yaml文件前，我们需要先准备nginx的docker镜像：
```
# 必须在worker节点上pull，因为master节点不允许调度业务容器
docker pull nginx:latest
```

pod-demo1.yaml：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: p-mynginx
spec:
  containers:
  - name: c-mynginx
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    ports:
    - name: nginxport
      containerPort: 80
```

执行资源清单文件：
```
kubectl apply -f pod-demo1.yaml
```

### 2.3 访问Pod

实际开发中，不推荐访问Pod，因为宿主机重启时，Pod的地址会发生变化，这里只是一个演示：
```
# 查找并显示pod的详细信息
kubectl get pods -o wide

# 显示结果
NAME                          READY   STATUS    RESTARTS   AGE    IP            NODE      NOMINATED NODE   READINESS GATES
p-mynginx-5f54f59897-7jrdr    1/1     Running   0          102s   10.244.3.10   worker1   <none>           <none>
p-mynginx-5f54f59897-mfngq    1/1     Running   0          102s   10.244.3.9    worker1   <none>           <none>

# 任意机器访问pod所在ip即可
curl http://10.244.3.10 
curl http://10.244.3.9
```

### 2.4 删除pod

使用命令行删除pod：
```
kubectl delete pods pod名称
```
同样，删除pod也可以使用资源清单进行删除。