# 使用minikube搭建单机Kubernetes
https://time.geekbang.org/column/article/529780

## 一、安装minikube
https://minikube.sigs.k8s.io/docs/
https://docs.docker.com/engine/install/debian/
```shell
#ARM64 Linux 安装
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-arm64
sudo install minikube-linux-arm64 /usr/local/bin/minikube && rm minikube-linux-arm64
minikube start

#kubectl 还提供了命令自动补全的功能，你还应该再加上“kubectl completion”：
source < (kubectl completion bash)
#如果有语法报错，可以试试下面的方法
kubectl completion bash > /tmp/kubectl-completion 
source /tmp/kubectl-completion

```

