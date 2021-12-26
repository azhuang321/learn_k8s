# replicaSet 
[官方文档](https://kubernetes.io/zh/docs/concepts/workloads/controllers/replicaset/)

```shell
kubectl create -f 7-1-replicaSet.yaml 
kubectl delete -f 7-1-replicaSet.yaml 
```

# deployment
[官方文档](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/)

```shell
kubectl create -f 7-2-deployment.yaml
kubectl get all

[root@master chapter7]# vim 7-2-deployment.yaml 
[root@master chapter7]# kubectl apply -f 7-2-deployment.yaml 
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment.apps/nginx configured

```


# StatefulSets
[官方文档](https://kubernetes.io/zh/docs/concepts/workloads/controllers/statefulset/)

```shell
kubectl create -f stateful.yaml
kubectl delete -f stateful.yaml
```

# DaemonSet
[官方文档](https://kubernetes.io/zh/docs/concepts/workloads/controllers/daemonset/)

# Job
[官方文档](https://kubernetes.io/zh/docs/concepts/workloads/controllers/job/)