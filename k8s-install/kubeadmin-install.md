# 使用kubeadmin搭建多节点的Kubernetes集群
https://time.geekbang.org/column/article/534762

## 一、安装前的准备工作
```shell
#1. 修改“/etc/hostname  比如 Master 节点就叫 master，Worker 节点就叫 worker
sudo vi /etc/hostname
#2. Docker 的配置修改，重启docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
#为了让 Kubernetes 能够检查、转发网络流量
#3. 修改 iptables 的配置，启用“br_netfilter”模块：
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward=1 # better than modify /etc/sysctl.conf
EOF

sudo sysctl --system
#4. 修改“/etc/fstab”，关闭 Linux 的 swap 分区，提升 Kubernetes 的性能
sudo swapoff -a
sudo sed -ri '/\sswap\s/s/^#?/#/' /etc/fstab
```
## 二、安装 kubeadm
```shell
sudo apt install -y apt-transport-https ca-certificates curl
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
#可以不用阿里的源
#cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
#deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
#EOF
sudo apt update
#指定版本号
sudo apt install -y kubeadm=1.23.3-00 kubelet=1.23.3-00 kubectl=1.23.3-00
#查看版本号
kubeadm version
kubectl version --client
#锁定这三个软件的版本，避免意外升级导致版本错误
sudo apt-mark hold kubeadm kubelet kubectl
```

## 三、使用 kubeadm 部署 Kubernetes 集群
```shell
#1. 安装 Master 节点
sudo kubeadm init \
	--pod-network-cidr=10.244.0.0/16 \
	--service-cidr=10.96.0.0/12 \
	--apiserver-advertise-address=10.3.31.213 \
	--kubernetes-version=v1.23.3
#scp  root@10.3.31.213:~/.kube/config ~/.kube/
#2. 安装 Calico 或者 Flannel网络插件 
#------Calico
wget https://projectcalico.docs.tigera.io/manifests/calico.yaml
kubectl apply -f calico.yaml
kubectl get pod -n kube-system -o wide
kubectl get node
#------Flannel
wget https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
vim kube-flannel.yml
#  net-conf.json: |
#    {
#      "Network": "10.244.0.0/16",
#      "Backend": {
#        "Type": "vxlan"
#      }
#    }
kubectl apply -f kube-flannel.yml
kubectl get pod  -n kube-flannel -o wide
kubectl get node
#3. 安装 Worker 节点
```bash
kubeadm join 10.3.31.213:6443 --token 8n5278.27j359p5mpq5m36p \
	--discovery-token-ca-cert-hash sha256:c1ae22e541661d265adb2c913000aaecc1f9154b585db9d39e9e6e8713145057
```