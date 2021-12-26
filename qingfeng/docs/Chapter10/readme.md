# jfrog container registry 搭建教程

```shell
rm -rf /etc/artifactory \
mkdir -p /etc/artifactory/var/etc/ \ 
cd /etc/artifactory/var/etc/ \
touch system.yaml \
chown -R 1030:1030 /etc/artifactory/var/ \
chmod -R 777 /etc/artifactory/var/ 
docker run --name artifactory-jcr -d -v /etc/artifactory/var/:/opt/jfrog/artifactory/var -p 8081:8081 -p 8082:8082 --privileged=true docker.bintray.io/jfrog/artifactory-jcr:latest

docker tag registry.cn-beijing.aliyuncs.com/qingfeng666/kubeblog:1.0 art.local:8081/docker-test/kubeblog:1.0

docker push art.local:8081/docker-test/kubeblog:1.0

kubectl create secret generic mysql-password-test --from-literal=MYSQL_PASSWORD_TEST=password

```





[官方文档](https://kubernetes.io/zh/docs/tasks/configure-pod-container/pull-image-private-registry/)



# 生成密钥(chapter11)

kubectl create secret docker-registry regcred-local --docker-server=art.local --docker-username=admin --docker-password=123123123 --docker-email=azhuang321@gmail.com
