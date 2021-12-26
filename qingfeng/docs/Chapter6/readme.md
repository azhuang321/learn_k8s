# 6-1 service对象&实践
[官方文档](https://kubernetes.io/zh/docs/concepts/services-networking/service/)
- 查看服务
`kubectl get service`

# 6-2 创建 Service 连接到应用
## 在集群中暴露 Pod

创建一个 Nginx Pod，声明它具有一个容器端口80：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80


```

这使得可以从集群中任何一个节点来访问它。检查节点，该 Pod 正在运行：

```shell
[root@master chapter6]# kubectl create -f 6-2nginx-deplyment.yaml 
deployment.apps/my-nginx created
[root@master chapter6]# kubectl get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/my-nginx-768b896b9f-55ddl   1/1     Running   0          11s
pod/my-nginx-768b896b9f-vb4wz   1/1     Running   0          11s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.1.0.1     <none>        443/TCP   19h

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx   2/2     2            2           11s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/my-nginx-768b896b9f   2         2         2       11s
[root@master chapter6]# kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
my-nginx-768b896b9f-55ddl   1/1     Running   0          17s
my-nginx-768b896b9f-vb4wz   1/1     Running   0          17s
[root@master chapter6]# kubectl get pods --output=wide
NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE    NOMINATED NODE   READINESS GATES
my-nginx-768b896b9f-55ddl   1/1     Running   0          52s   10.244.2.4    node2   <none>           <none>
my-nginx-768b896b9f-vb4wz   1/1     Running   0          52s   10.244.1.19   node1   <none>           <none>

[root@master chapter6]# kubectl get pods -l run=my-nginx -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP            NODE    NOMINATED NODE   READINESS GATES
my-nginx-768b896b9f-55ddl   1/1     Running   0          3m51s   10.244.2.4    node2   <none>           <none>
my-nginx-768b896b9f-vb4wz   1/1     Running   0          3m51s   10.244.1.19   node1   <none>           <none>

```

检查 Pod 的 IP 地址：

```shell
kubectl get pods -l run=my-nginx -o wide | gawk '{print $6}'
    podIP: 10.244.3.4
    podIP: 10.244.2.5

```

此时能够通过 ssh 登录到集群中的任何一个节点上，使用 curl 也能调通所有 IP 地址。 需要注意的是，容器不会使用该节点上的 80 端口，也不会使用任何特定的 NAT 规则去路由流量到 Pod 上。 这意味着可以在同一个节点上运行多个 Pod，使用相同的容器端口，并且可以从集群中任何其他的 Pod 或节点上使用 IP 的方式访问到它们。

## 创建 Service

Kubernetes Service 从逻辑上定义了运行在集群中的一组 Pod，这些 Pod 提供了相同的功能。 当每个 Service 创建时，会被分配一个唯一的 IP 地址（也称为 clusterIP）。 这个 IP 地址与一个 Service 的生命周期绑定在一起，当 Service 存在的时候它也不会改变。 可以配置 Pod 使它与 Service 进行通信，Pod 知道与 Service 通信将被自动地负载均衡到该 Service 中的某些 Pod 上。

可以使用  `kubectl expose`  命令为 2个 Nginx 副本创建一个 Service：

```shell
kubectl expose deployment/my-nginx
service/my-nginx exposed

```

这等价于使用  `kubectl create -f 6-2nginx-deplyment.yaml `  命令创建，对应如下的 yaml 文件：
nginx-svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    run: my-nginx

```

上述规约将创建一个 Service，对应具有标签  `run: my-nginx`  的 Pod，目标 TCP 端口 80， 并且在一个抽象的 Service 端口（`targetPort`：容器接收流量的端口；`port`：抽象的 Service 端口，可以使任何其它 Pod 访问该 Service 的端口）上暴露。 查看你的 Service 资源:

```shell
[root@master chapter6]# kubectl get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/my-nginx-768b896b9f-55ddl   1/1     Running   0          19m
pod/my-nginx-768b896b9f-vb4wz   1/1     Running   0          19m

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.1.0.1     <none>        443/TCP   19h

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx   2/2     2            2           19m

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/my-nginx-768b896b9f   2         2         2       19m
[root@master chapter6]# kubectl create -f 6-2nginx-service.yaml 
service/my-nginx created
[root@master chapter6]# kubectl get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/my-nginx-768b896b9f-55ddl   1/1     Running   0          19m
pod/my-nginx-768b896b9f-vb4wz   1/1     Running   0          19m

NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.1.0.1      <none>        443/TCP   19h
service/my-nginx     ClusterIP   10.1.157.62   <none>        80/TCP    3s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-nginx   2/2     2            2           19m

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/my-nginx-768b896b9f   2         2         2       19m

#正如前面所提到的，一个 Service 由一组 backend Pod 组成。这些 Pod 通过  `endpoints`  暴露出来。 Service Selector 将持续观察，结果被 POST 到一个名称为  `my-nginx`  的 Endpoint 对象上。 当 Pod 终止后，它会自动从 Endpoint 中移除，新的能够匹配上 Service Selector 的 Pod 将自动地被添加到 Endpoint 中。 检查该 Endpoint，注意到 IP 地址与在第一步创建的 Pod 是相同的。
[root@master chapter6]# kubectl describe service my-nginx
Name:              my-nginx
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          run=my-nginx
Type:              ClusterIP
IP:                10.1.157.62
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.19:80,10.244.2.4:80
Session Affinity:  None
Events:            <none>
```

```shell
kubectl get ep my-nginx

NAME       ENDPOINTS                     AGE
my-nginx   10.244.2.5:80,10.244.3.4:80   1m
```
现在，能够从集群中任意节点上使用 curl 命令请求 Nginx Service  `<CLUSTER-IP>:<PORT>`  。 注意 Service IP 完全是虚拟的，它从来没有走过网络。

## 访问 Service
[官方文档](https://kubernetes.io/zh/docs/concepts/services-networking/dns-pod-service/)  
Kubernetes支持两种查找服务的主要模式: 环境变量和DNS。 前者开箱即用，而后者则需要CoreDNS

### 环境变量
当 Pod 在 Node 上运行时，kubelet 会为每个活跃的 Service 添加一组环境变量。 这会有一个顺序的问题。想了解为何，检查正在运行的 Nginx Pod 的环境变量（Pod 名称将不会相同）：

```shell
[root@master chapter6]# kubectl exec -it my-nginx-768b896b9f-jspdl printenv | grep MY_NGINX
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
MY_NGINX_PORT_80_TCP_ADDR=10.1.157.62
MY_NGINX_SERVICE_HOST=10.1.157.62
MY_NGINX_PORT_80_TCP_PORT=80
MY_NGINX_SERVICE_PORT=80
MY_NGINX_PORT_80_TCP_PROTO=tcp
MY_NGINX_PORT=tcp://10.1.157.62:80
MY_NGINX_PORT_80_TCP=tcp://10.1.157.62:80
```

# 创建集群外部访问的服务
[官方文档](https://kubernetes.io/zh/docs/tutorials/stateless-application/expose-external-ip-address/)
解决dns 不通的问题: kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml


# ingress 实践

[官方文档](https://kubernetes.io/zh/docs/concepts/services-networking/ingress/)  
`kubectl create -f 6-6-ingress-nginx-controller.yaml`

```shell
kubectl create deployment web --image=registry.cn-beijing.aliyuncs.com/qingfeng666/hello-app:1.0
kubectl expose deployment web --type=NodePort --port=8080
kubectl create -f 6-6-example-ingress.yaml
```





