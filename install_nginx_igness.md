每次增加igress
这个东西就监听api server
都会在ngixn config中增加转发规则
cat /etc/nginx/nginx.conf
## 官网安装
https://github.com/kubernetes/ingress-nginx/blob/master/docs/deploy/index.md

nginxinc 安装
https://github.com/nginxinc/kubernetes-ingress/blob/master/install/service/nodeport.yaml

ingress 配置相关参数
https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/annotations.md

## HELM 安装
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


增加Ingress
```
You can watch the status by running 'kubectl --namespace default get services -o wide -w nginx-ingress-controller'

An example Ingress that makes use of the controller:

  apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls

cat /root/certs/dashboard.key | base64 | tr -d '\n'
```
