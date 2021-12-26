# 5-1创建一个nginx pod
```shell
kubectl create -f nginx.yaml

#查看pod
kubectl get po

#查看详细信息
kubectl describe pod my-nginx

#进入容器
kubectl exec -it my-nginx sh
```

# 5-2创建pod
```shell
kubectl create -f hello-job.yaml  #创建job
kubectl get job   #查看job
kubectl get po    #查看pod
kubectl get po -w  #watch pod
kubectl logs hello-swldv  #查看日志
kubectl delete job hello  #清空job
kubectl get po
```

# 5-3容器生命周期提供事件处理
[官方文档](https://kubernetes.io/zh/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/)

# 5-4 init容器
[官方文档](https://kubernetes.io/zh/docs/concepts/workloads/pods/init-containers/)
```shell
[root@master ~]# kubectl apply -f iniPod.yaml 
pod/myapp created
[root@master ~]# kubectl get po
NAME       READY   STATUS     RESTARTS   AGE
my-nginx   1/1     Running    0          141m
myapp      0/1     Init:0/1   0          2s
[root@master ~]# kubectl get po -w
NAME       READY   STATUS     RESTARTS   AGE
my-nginx   1/1     Running    0          141m
myapp      0/1     Init:0/1   0          7s
myapp      0/1     Init:0/1   0          10s
^C[root@master ~]# kubectl logs myapp
Mon Dec 20 04:47:53 UTC 2021
[root@master ~]# kubectl logs myapp -c init-container  # 查看pod内容器的日志
Mon Dec 20 04:47:42 UTC 2021
```

# 5-5 容器探针
[官方文档](https://kubernetes.io/zh/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)
[官方例子](https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
```shell
[root@master ~]# vim liveness.yaml
[root@master ~]# kubectl apply -f liveness.yaml 
pod/liveness-exec created
[root@master ~]# kubectl get po -w
NAME            READY   STATUS    RESTARTS   AGE
liveness-exec   1/1     Running   0          11s
my-nginx        1/1     Running   0          154m
myapp           1/1     Running   0          12m
liveness-exec   1/1     Running   1          78s
[root@master ~]# kubectl describe po liveness-exec
...
...
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  2m31s                default-scheduler  Successfully assigned default/liveness-exec to node1
  Normal   Pulled     2m27s                kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 3.103417087s
  Normal   Pulled     74s                  kubelet            Successfully pulled image "k8s.gcr.io/busybox" in 1.374195808s
  Normal   Created    74s (x2 over 2m27s)  kubelet            Created container liveness
  Normal   Started    73s (x2 over 2m27s)  kubelet            Started container liveness
  Warning  Unhealthy  30s (x6 over 115s)   kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    30s (x2 over 105s)   kubelet            Container liveness failed liveness probe, will be restarted
  Normal   Pulling    0s (x3 over 2m30s)   kubelet            Pulling image "k8s.gcr.io/busybox"
```

# 5-6 为容器设置启动时要执行的命令和参数
[官方文档](https://kubernetes.io/zh/docs/tasks/inject-data-application/define-command-argument-container/)


# 5-8 为容器定义相互依赖的环境变量
当创建一个 Pod 时，你可以为运行在 Pod 中的容器设置相互依赖的环境变量。 设置相互依赖的环境变量，你就可以在配置清单文件的  `env`  的  `value`  中使用 $(VAR_NAME)。

在本练习中，你会创建一个单容器的 Pod。 此 Pod 的配置文件定义了一个已定义常用用法的相互依赖的环境变量。 下面是 Pod 的配置清单：


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dependent-envars-demo
spec:
  containers:
    - name: dependent-envars-demo
      args:
        - while true; do echo -en '\n'; printf UNCHANGED_REFERENCE=$UNCHANGED_REFERENCE'\n'; printf SERVICE_ADDRESS=$SERVICE_ADDRESS'\n';printf ESCAPED_REFERENCE=$ESCAPED_REFERENCE'\n'; sleep 30; done;
      command:
        - sh
        - -c
      image: busybox
      env:
        - name: SERVICE_PORT
          value: "80"
        - name: SERVICE_IP
          value: "172.17.0.1"
        - name: UNCHANGED_REFERENCE
          value: "$(PROTOCOL)://$(SERVICE_IP):$(SERVICE_PORT)"
        - name: PROTOCOL
          value: "https"
        - name: SERVICE_ADDRESS
          value: "$(PROTOCOL)://$(SERVICE_IP):$(SERVICE_PORT)"
        - name: ESCAPED_REFERENCE
          value: "$$(PROTOCOL)://$(SERVICE_IP):$(SERVICE_PORT)"

```

1.  依据清单创建 Pod：
    
    ```shell
    kubectl apply -f https://k8s.io/examples/pods/inject/dependent-envars.yaml
    
    ```
    
    ```
    pod/dependent-envars-demo created
    
    ```
    
2.  列出运行的 Pod：
    
    ```shell
    kubectl get pods dependent-envars-demo
    
    ```
    
    ```
    NAME                      READY     STATUS    RESTARTS   AGE
    dependent-envars-demo     1/1       Running   0          9s
    
    ```
    
3.  检查 Pod 中运行容器的日志：
    
    ```shell
    kubectl logs pod/dependent-envars-demo
    
    ```
    
    ```
    
    UNCHANGED_REFERENCE=$(PROTOCOL)://172.17.0.1:80
    SERVICE_ADDRESS=https://172.17.0.1:80
    ESCAPED_REFERENCE=$(PROTOCOL)://172.17.0.1:80
    
    ```
    

如上所示，你已经定义了  `SERVICE_ADDRESS`  的正确依赖引用，  `UNCHANGED_REFERENCE`  的错误依赖引用， 并跳过了  `ESCAPED_REFERENCE`  的依赖引用。

如果环境变量被引用时已事先定义，则引用可以正确解析， 比如  `SERVICE_ADDRESS`  的例子。

当环境变量未定义或仅包含部分变量时，未定义的变量会被当做普通字符串对待， 比如  `UNCHANGED_REFERENCE`  的例子。 注意，解析不正确的环境变量通常不会阻止容器启动。

`$(VAR_NAME)`  这样的语法可以用两个  `$`  转义，既：`$$(VAR_NAME)`。 无论引用的变量是否定义，转义的引用永远不会展开。 这一点可以从上面  `ESCAPED_REFERENCE`  的例子得到印证。


# 5-9 容器的资源限制
[官方文档](https://kubernetes.io/zh/docs/tasks/configure-pod-container/assign-memory-resource/)

查看pod的yaml文件
`kubectl get pod myapp -o yaml`

# 5-10 节点亲和性把 Pods 分配到节点
[官方文档](https://kubernetes.io/zh/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/)

# 5-11 ConfigMap
[官方文档](https://kubernetes.io/zh/docs/concepts/configuration/configmap/)

# 5-12 root vs privileged 
```shell
[root@node2 ~]# docker run --rm -it busybox sh
/ # whoami
root
/ # id
uid=0(root) gid=0(root) groups=10(wheel)
/ # hostname
eb25e9c1c2a2
/ # sysctl kernel.hostname=attacker
sysctl: error setting key 'kernel.hostname': Read-only file system
/ # exit
[root@node2 ~]# docker run --rm -it --privileged busybox sh
/ # whoami
root
/ # id
uid=0(root) gid=0(root) groups=10(wheel)
/ # hostname
17db4b0155ac
/ # sysctl kernel.hostname=attacker
kernel.hostname = attacker
/ # hostname
attacker
/ # exit
```

# 5-12 为 Pod 或容器配置安全性上下文
[官方文档](https://kubernetes.io/zh/docs/tasks/configure-pod-container/security-context/)
