# nginx-ingress-controller
```bash
 kubectl port-forward -n nginx-ingress ngx-kic-dep-65469dcd4d-tx5z6  8080:80 &
 curl --resolve ngx.test:8080:127.0.0.1 http://ngx.test:8080
```
* 可以修改 /etc/hosts 来手工添加域名解析，
* 也可以使用 --resolve 参数，指定域名的解析规则， 
* 比如在这里我就把“ngx.test”强制解析到“127.0.0.1”，
* 也就是被 kubectl port-forward 转发的本地地址


# kong-ingress-controller

## 安装 kong-ingress-controller
```bash
wget https://github.com/Kong/kubernetes-ingress-controller/archive/refs/tags/v2.7.0.tar.gz
kubectl apply -f all-in-one-dbless.yaml
kubectl get pod -n kong
kubectl get svc -n kong
curl 10.3.31.213:31863  -i
```
从 curl 获取的响应结果可以看到， Kong Ingress Controller 2.7 内部使用的 Kong 版本是 3.0.1，
因为现在我们没有为它配置任何 Ingress 资源，所以返回了状态码 404，这是正常的。


## 使用 Kong Ingress Controller
```bash
kubectl apply -f ngx-cm.yml
kubectl apply -f ngx-dep.yml
kubectl apply -f ngx-svc.yml
kubectl apply -f kong-ing.yml
kubectl apply -f kong-ink.yml
kubectl apply -f kong-kic.yml
kubectl get pod,svc -n kong
curl 10.3.31.213:31823 -H 'host: kong.test' -v
curl 10.3.31.213:31823 -H 'host: kong.dev' -v
curl 10.3.31.213:31823 -H 'host: kong.ops' -v
curl 10.3.31.213:31823 -H 'host: abc.ccc' -v

kubectl api-resources  | grep kong
kubectl explain kongplugins
kubectl apply -f kong-crd.yml
curl 10.3.31.213:31823 -H 'host: kong.test' -v
```

# kong-ingress-controller 小结
Kubernetes 管理集群进出流量的工具：Kong Ingress Controller:
* Kong Ingress Controller 的底层内核仍然是 Nginx，但基于 OpenResty 和 LuaJIT，实现了对路由的完全动态管理，不需要 reload。
* 使用“无数据库”的方式可以非常简单地安装 Kong Ingress Controller，它是一个由两个容器组成的 Pod。
* Kong Ingress Controller 支持标准的 Ingress 资源，但还使用了 annotation 和 CRD 提供更多的扩展增强功能，特别是插件，可以灵活地加载或者拆卸，实现复杂的流量管理策略。
