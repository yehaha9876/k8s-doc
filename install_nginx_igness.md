每次增加igress
这个东西就监听api server
都会在ngixn config中增加转发规则
cat /etc/nginx/nginx.conf

安装过程
```
helm fetch stable/nginx-ingress 
tar xzf nginx-ingress-0.20.1.tgz
```

helm install --name nginx-ingress nginx-ingress-0.20.1

kubectl set image deployment nginx-ingress-default-backend nginx-ingress-default-backend=registry.cn-hangzhou.aliyuncs.com/test_k8s/defaultbackend:1.3


删除：
helm delete --purge nginx-ingress

curl --resolve k8s.gzky.com:80:192.168.3.60 k8s.gzky.com:80/demo-test/person/list-all

参考文档：
https://kubernetes.github.io/ingress-nginx/deploy/
https://github.com/kubernetes/ingress-nginx
https://blog.csdn.net/aixiaoyang168/article/details/78485581?locationNum=5&fps=1
