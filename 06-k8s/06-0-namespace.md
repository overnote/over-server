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
