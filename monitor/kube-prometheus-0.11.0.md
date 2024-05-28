
https://github.com/prometheus-operator/kube-prometheus/
https://prometheus.io/docs/introduction/overview/
https://prometheus.io/
https://time.geekbang.org/column/article/550598
```bash
wget https://github.com/prometheus-operator/kube-prometheus/archive/refs/tags/v0.11.0.tar.gz
kubectl create -f manifests/setup
kubectl create -f manifests
kubectl get pod -n monitoring -o wide
kubectl get svc -n monitoring -o wide

```

修改 prometheus-service.yaml、grafana-service.yaml。
这两个文件定义了 Prometheus 和 Grafana 服务对象，我们可以给它们添加 type: NodePort，
这样就可以直接通过节点的 IP 地址访问（也可以配置成 Ingress）。

**Prometheus** 的 Web 界面比较简单，通常只用来调试、测试，不适合实际监控
加上端口号“30827”，我们就能看到 Prometheus 自带的 Web 界面，
Web 界面上有一个查询框，可以使用 PromQL 来查询指标，生成可视化图表，
比如在这个截图里我就选择了“**node_memory_Active_bytes**”这个指标，意思是当前正在使用的内存容量。


**Grafana**，访问节点的端口“30358”（我这里是“http://192.168.10.210:30358”），它会要求你先登录，默认的用户名和密码都是“admin”
Grafana 内部已经预置了很多强大易用的仪表盘，你可以在左侧菜单栏的“Dashboards - Browse”里任意挑选一个：
比如我选择了“Kubernetes / Compute Resources / Namespace (Pods)”这个仪表盘，就会出来一个非常漂亮图表，
比 Metrics Server 的 kubectl top 命令要好看得多，各种数据一目了然
