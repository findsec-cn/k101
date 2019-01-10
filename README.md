# ubernetes 技术实践.基础篇

## 容器打包
docker build -t ist0ne/hello:v1 .
docker login
docker push

## 启动一个单节点nginx服务
kubectl run nginx --image=nginx:1.15

## 访问Dashboard
kubectl proxy
http://localhost:8001/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy/#!/overview?namespace=default

## 获取pods
kubectl get pods

## 启动工具容器进行访问测试
kubectl run -ti busybox --image=busybox --restart=Never -- sh
## 测试不能获取到服务
wget http://nginx

## 为部署创建服务
kubectl expose deployment nginx --port 80

## 获取部署列表
kubectl get svc

## 启动工具容器进行访问测试
kubectl run -ti busybox --image=busybox --restart=Never -- sh
## 测试能获取到服务
wget http://nginx

## 创建nginx 配置
kubectl create configmap nginx-conf --from-file=nginx/hello.conf

## 部署flask 应用
kubectl create -f pods/nginx-flask-app.yaml
kubectl get pods nginx

## 更新版本
kubectl replace --force -f pods/nginx-flask-app.yaml

## 访问测试
kubectl port-forward nginx 10080:80
curl http://127.0.0.1:10080

## 查看容器日志
kubectl logs -c nginx nginx
kubectl logs -c hello nginx

kubectl exec -ti nginx -c hello bash


## 项目部署
### 获取pods
kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
auth-7fd44dbc5b-klz9k       1/1     Running   0          108s
frontend-84794db478-2wqrk   1/1     Running   0          108s
hello-9944bc998-wgq88       1/1     Running   0          107s

### 登录虚拟机
minikube ssh

## 拷贝ca.pem到虚拟机上

## 访问非授权接口返回成功
curl --cacert ./ca.pem https://127.0.0.1:30443/hello
{"message":"Hello"}

## 访问非授权接口返回失败
curl --cacert ./ca.pem https://127.0.0.1:30443/secure
authorization failed

## 获取接口JWT Token
curl --cacert ./ca.pem -u user https://127.0.0.1:30443/login
Enter host password for user 'user':password
{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InVzZXJAZXhhbXBsZS5jb20iLCJleHAiOjE1NDcyNzg4NjcsImlhdCI6MTU0NzAxOTY2NywiaXNzIjoiYXV0aC5zZXJ2aWNlIiwic3ViIjoidXNlciJ9.bT3flMe_VywoFkGCFt08Tw0fxytKZblj8lBHNVLYC6U"}

## 使用Token访问接口成功返回
curl --cacert ./ca.pem -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InVzZXJAZXhhbXBsZS5jb20iLCJleHAiOjE1NDcyNzg4NjcsImlhdCI6MTU0NzAxOTY2NywiaXNzIjoiYXV0aC5zZXJ2aWNlIiwic3ViIjoidXNlciJ9.bT3flMe_VywoFkGCFt08Tw0fxytKZblj8lBHNVLYC6U' https://127.0.0.1:30443/secure


