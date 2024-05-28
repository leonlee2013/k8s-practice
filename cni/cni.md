# Flannel
```bash
kubectl create deploy ngx-dep --image=nginx:alpine --replicas=3
kubectl get pod -o wide

kubectl exec ngx-dep-bfbb5f64b-9hnrd  -- ip addr
#宿主机执行
ip addr 
apt-get install bridge-utils
brctl show

route

ip  neighbor | grep 10.244.2
bridge fdb | grep flannel
route
```


# Calico
```bash
wget https://projectcalico.docs.tigera.io/manifests/calico.yaml
kubectl apply -f calico.yaml
kubectl get pod -n kube-system 
kubectl delete deploy ngx-dep 
kubectl create deploy ngx-dep --image=nginx:alpine --replicas=3
kubectl get pod -o wide
kubectl exec ngx-dep-bfbb5f64b-mwb7p  -- ip addr

#宿主机执行
ip addr 
brctl show
route
```