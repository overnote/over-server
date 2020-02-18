## 四 Service

### 4.1 Service概述

在第三节中，通过Controller可以创建一个应用，但是此时Pod的状态不是人为控制的，Pod的IP在创建时自动分配，如果Pod被误删，Controller重新拉起一个新Pod时，Pod的地址会发生变化，即应用本身也需要更换IP地址，这肯定在企业环境是不合适的。   

k8s为了解决上述问题,引入了Service，无论Pod的IP如何变化，其所在的service是不会发生改变的。Service并不是微服务概念中的实体服务，只是一个iptables或者ipvs的转发规则。  

通过Sercice可以为客户端提供访问Pod方法，在k8s内部，Service通过Pod标签与Pod进行关联。    

Service也有很多种类型：
- ClusterIP：默认类型，分配一个集群内部可以访问的虚拟IP
- NodePort：在每个Node上分配一个端口作为外部访问入口
- LoadBalancer：工作在特定的Cloud Provider上，例如AWS，OpenStack
- ExternalName：表示把集群外部的服务引入到集群内部中来，实现集群内部pod和集群外部服务进行通信

Service常用参数：
- port：访问service使用的端口
- targetPort：Pod中容器的端口
- NodePort：通过Node实现外网用户访问k8s集群内service的端口（30000-32767）

通过命令行创建Service：
```
# 在上一节已经创建了一个应用
kubectl run nginx_app1 --image=nginx:latest --image-pull-policy=IfNotPresent --replicas=2

# 此时可以直接访问其Pod，但是不推荐
kubectl get pods -o wide        # 查看pod的id

NAME                          READY   STATUS    RESTARTS   AGE   IP           NODE      NOMINATED NODE   READINESS GATES
nginx-app1-7c49567bb6-858f5   1/1     Running   0          75m   10.244.3.7   worker1   <none>           <none>
nginx-app1-7c49567bb6-qtks6   1/1     Running   0          75m   10.244.3.8   worker1   <none>           <none>

curl http://10.244.3.7          # 直接访问pod

# 让应用与service关联。target-port是容器暴露的端口，port是访问service的端口
kubectl expose deployment.apps nginx_app1 --type=ClusterIP --target-port=80 --port=8011

# 查看service
kubectl get service             # 删除方法也是 kubectl delete service service名

# 通过service访问pod
curl http://10.1.209.119:8011
```

Service与Pod是通过断点关联的：
```
kubectl get endpoints
```

### 4.2 通过资源清单创建Service

service的资源清单既可以和Deployment写在一起，也可以分开，这里分开书写。  

资源清单文件 ser-demo1.yaml（注意---分隔符）：
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: nginx-app2
spec:
  replicas: 2
  selector:             
    matchLabels:
      env: nginx-pl
  template:
    metadata:
      name: nginx-p
      labels:
        env: nginx-pl
    spec:
      containers:
      - name: nginx-c
        image: nginx:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata: 
  name: nginx-srv2
spec: 
  type: ClusterIP   # 修改为NodePort,即可实现集群外访问
  ports: 
  - protocol: TCP
    port: 8011
    targetPort: 80  
  selector: # 与pod的标签关联
    env: nginx-pl
```

使用：
```
# 添加  删除也是相应的 delete
kubectl apply -f ser-demo1.yaml

# 查看service，删除方法也是 kubectl delete service service名
kubectl get service              

# 通过service访问pod
curl http://10.1.210.41:8011

# 查看关联
kubectl get endpoints
```

### 4.3 NodePort类型service 支持集群外访问

如果要指定集群外部可以访问,则service为:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: nginx-app3
spec:
  replicas: 2
  selector:             
    matchLabels:
      env: nginx-pl3
  template:
    metadata:
      name: nginx-p
      labels:
        env: nginx-pl3
    spec:
      containers:
      - name: nginx-c
        image: nginx:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata: 
  name: nginx-srv3
spec: 
  type: NodePort 
  ports: 
  - protocol: TCP
    port: 8011
    targetPort: 80
    nodePort: 30001  # 集群外端口，不设定也会自动设定
  selector: 
    env: nginx-pl3
```

这样服务会在宿主机集群开启30001 端口监听，此时外部就可以对集群进行访问了！

### 4.4 演示 service 的负载均衡

```
# 查看pod
kubectl get pods

NAME                         READY   STATUS    RESTARTS   AGE
nginx-app2-86b4c78df-ck8m2   1/1     Running   0          2m50s
nginx-app2-86b4c78df-lhrbs   1/1     Running   0          2m50s

# 进入pod1，并修改一些内容
kubectl exec -it nginx-app2-86b4c78df-ck8m2 sh
cd /usr/share/nginx/html
ls -l
echo "pod1......" > index.html
exit

# 进入pod2，并修改一些内容
kubectl exec -it nginx-app2-86b4c78df-lhrbs  sh
cd /usr/share/nginx/html
ls -l
echo "pod2......" > index.html
exit

# 重复访问服务查看负载均衡情况
curl http://10.1.210.41:8011
```