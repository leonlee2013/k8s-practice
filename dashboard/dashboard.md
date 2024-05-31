
# helm 安装
https://github.com/kubernetes/dashboard?tab=readme-ov-file
https://github.com/kubernetes/dashboard/issues/8765
```bash
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard --set kong.admin.tls.enabled=false

#远程访问
kubectl -n kubernetes-dashboard port-forward --address 10.3.31.213 svc/kubernetes-dashboard-kong-proxy 8443:443

#获取token
kubectl apply -f admin-user.yaml
#拷贝 token
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

https://10.3.31.213:8443
```

# helm 卸载
```bash
helm list --all-namespaces
helm uninstall kubernetes-dashboard --namespace kubernetes-dashboard
helm list --all-namespaces
```

# yml部署dashboard
```bash
openssl req -x509 -days 365 -out k8s.test.crt -keyout k8s.test.key \
  -newkey rsa:2048 -nodes -sha256 \
    -subj '/CN=k8s.test' -extensions EXT -config <( \
       printf "[dn]\nCN=k8s.test\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:k8s.test\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```
生成的是一个 X509 格式的证书，有效期 365 天，私钥是 RSA2048 位，摘要算法是 SHA256，签发的网站是“k8s.test”。
运行命令行后会生成两个文件，一个是证书“k8s.test.crt”，另一个是私钥“k8s.test.key”，我们需要把这两个文件存入 Kubernetes 里供 Ingress 使用
```bash


kubectl create secret tls dash-tls -n kubernetes-dashboard --cert=k8s.test.crt --key=k8s.test.key --dry-run=client -o yaml  > cert.yml
kubectl create ing dash-ing --rule="k8s.test/=kubernetes-dashboard:443" --class=dash-ink -n kubernetes-dashboard --dry-run=client -o yaml
kubectl get ingressclasses  -n  kubernets-dashboard
kubectl get ingress  -n  kubernetes-dashboard
kubectl get pod,svc  -n   nginx-ingress 
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```
测试域名“k8s.test”加上域名解析（修改 /etc/hosts），
在浏览器里输入网址“https://k8s.test:30443”访问 Dashboard