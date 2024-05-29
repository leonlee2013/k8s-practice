# 测试方法

```bash
 kubectl port-forward -n nginx-ingress ngx-kic-dep-65469dcd4d-tx5z6  8080:80 &
 curl --resolve ngx.test:8080:127.0.0.1 http://ngx.test:8080
```
* 可以修改 /etc/hosts 来手工添加域名解析，
* 也可以使用 --resolve 参数，指定域名的解析规则， 
* 比如在这里我就把“ngx.test”强制解析到“127.0.0.1”，
* 也就是被 kubectl port-forward 转发的本地地址