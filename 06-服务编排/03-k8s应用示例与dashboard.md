## 一 示例 使用k8s部署一个nginx

快速部署一个Nginx服务器：
```
# 部署
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort

# 查看服务端口
kubectl get pod,svc     # 会显示nginx的端口号为 80:32395

# 访问地址 http://node节点ip:80端口映射的端口号 
http://192.168.216.101:32395 
```


此时已经可以方便的做扩容了：
```
# 扩容nginx为3个副本
kubectl scale deployment nginx --replicas=3

# 查看扩容情况，并发性此时也上升了三倍
kubectl get pods
```

## 二 部署图形界面 Dashboard
```
# dashboard的yaml国内是无法访问的，可以下载后修改地址
wget https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

vim kubernetes-dashboard.yaml
# 修改镜像地址为： lizhenliang/kubernetes-dashboard-amd64:v1.10.1

# 默认Dashboard只能集群内部访问，修改Service为NodePort类型，暴露到外部：
kind: Service
apiVersion: v1
metadata:
labels:
  k8s-app: kubernetes-dashboard
name: kubernetes-dashboard
namespace: kube-system
spec:
type: NodePort
ports:
  - port: 443
    targetPort: 8443
    nodePort: 30001
selector:
  k8s-app: kubernetes-dashboard

# 修改完毕后也应用yaml文件
kubectl apply -f kubernetes-dashboard.yaml

# 查看dashboard运行情况：
kubectl get pods -n kube-system
```

访问地址：https://节点IP:30001   。  

访问后需要输入账户或者令牌，创建账户（名为 myadmin）并绑定默认cluster-admin管理员集群角色：
```
kubectl create serviceaccount myadmin -n kube-system
kubectl create clusterrolebinding myadmin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/myadmin/{print $1}')
```

使用输出的token登录Dashboard，不过该UI用户体验很差(LOL)，使用命令即可完成常用操作。  