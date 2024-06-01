# Gateway API
## 一、安装 Gateway API
Gateway API 比较特殊，并不集成在 Kubernetes 内部，而是在外部以相对独立的方式开发实现的，
所以需要用 YAML 文件的形式来部署进 Kubernetes。
```bash
wget https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/standard-install.yaml
kubectl apply -f standard-install.yaml
kubectl api-resources | grep gateway
```
## 二、安装 Kong Ingress Controller
```shell
#Helm安装 https://helm.sh/docs/intro/install/
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
#添加远端仓库（Helm charts），用法和 yum、apt 也很类似
helm repo add kong https://charts.konghq.com
helm repo update
#查看远端仓库里可用的安装包
helm repo list
helm search repo kong
#使用命令 helm install 指定名字和安装包，再加上一些定制参数就可以安装 Kong Ingress Controller。
helm install \
    kong kong/ingress \
    -n kong \
    --create-namespace \
    --set gateway.env.router_flavor=expressions
    #“–set” 选项，启用 Kong Ingress Controller 的表达式路由，能够更好地支持 Gateway API。
kubectl get pod,svc  -n kong
curl -i $(minikube ip):30223
kubectl get gc -o wide
kubectl get gtw
```

## 三、 创建实验用的 Gateway Class 和 Gateway 对象
```shell
kubectl apply -f kong-gateway.yml
kubectl get gc -o wide
kubectl get gtw
```

## 四、 准备后端服务
```bash
kubectl apply -f backend.yml
sed 's/ngx/red/g'   backend.yml | kubectl apply -f -
sed 's/ngx/green/g' backend.yml | kubectl apply -f -
sed 's/ngx/blue/g'  backend.yml | kubectl apply -f -
sed 's/ngx/black/g' backend.yml | kubectl apply -f -
```

## 五、使用 Gateway API 配置路由

```shell    
kubectl apply -f routes.yml
kubectl get svc -n kong kong-gateway-proxy
kubectl get HTTPRoute -o wide

curl -i $(minikube ip):30223 -H 'host: gtw.test'
curl -i $(minikube ip):30223/hello -H 'host: gtw.ops'
curl -i $(minikube ip):30223 -H 'host: gtw.dev' -H 'area: north'

curl -i $(minikube ip):30223 -H 'host: gtw.io'
curl -i $(minikube ip):30223?user=admin -H 'host: gtw.io'
curl -i $(minikube ip):30223/leaf888 -H 'host: gtw.ai'

kubectl apply -f traffic.yml
kubectl get svc -n kong kong-gateway-proxy
kubectl get HTTPRoute -o wide
curl -i $(minikube ip):30223 -H 'host: canary.test'
curl -i $(minikube ip):30223/login -H 'host: canary.test'
curl -i $(minikube ip):30223 -H 'host: blue-green.test'


kubectl apply -f filters.yml
kubectl get svc -n kong kong-gateway-proxy
kubectl get HTTPRoute -o wide
curl -i $(minikube ip):30223 -H 'host: filter.test'

```