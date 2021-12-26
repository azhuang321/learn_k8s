# 安装helm
[官方文档](https://helm.sh/zh/docs/intro/install/)
```shell
wget https://get.helm.sh/helm-v3.7.2-linux-amd64.tar.gz
tar -zxvf helm-v3.7.2-linux-amd64.tar.gz 
```

# example
```shell
[root@master chapter12]# helm create nginx
Creating nginx
[root@master chapter12]# helm package nginx
Successfully packaged chart and saved it to: /root/chapter12/nginx-0.1.0.tgz
[root@master chapter12]# helm install nginx nginx-0.1.0.tgz
NAME: nginx
LAST DEPLOYED: Wed Dec 22 04:49:39 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:

1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=nginx,app.kubernetes.io/instance=nginx" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
[root@master chapter12]# kubectl get pod 
NAME                     READY   STATUS              RESTARTS   AGE
mysql-0                  1/1     Running             2          14h
nginx-764c6bcc78-m56qg   0/1     ContainerCreating   0          18s
[root@master chapter12]# helm ls
NAME 	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
nginx	default  	1       	2021-12-22 04:49:39.377014423 +0000 UTC	deployed	nginx-0.1.0	1.16.0     
[root@master chapter12]# helm delete nginx
release "nginx" uninstalled
```

helm package kubeblog kubeblog
helm install kubeblog kubeblog-0.1.2.tgz
kubectl get all


## helm的应用
上传,与回滚
helm repo update
helm upgrade kubeblog helm/kubeblog --version 0.1.1

helm rollback kubeblog 1

## helm 命名空间环境隔离
helm install kubeblog kubeblog-0.1.2.tgz -n test # 部署到test命名空间下

## 使用不同配置文件
helm install kubeblog kubeblog-0.1.2.tgz -f kubeblog/values-dev.yaml -n test 

## jfrog的应用

