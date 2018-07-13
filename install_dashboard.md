文档
https://github.com/kubernetes/dashboard/wiki/Certificate-management
https://github.com/kubernetes/dashboard/wiki/Installation#recommended-setup

创建证书
cd certs/
ls
openssl genrsa -des3 -passout pass:x -out dashboard.pass.key 2048
openssl rsa -passin pass:x -in dashboard.pass.key -out dashboard.key
rm dashboard.pass.key
openssl req -new -key dashboard.key -out dashboard.csr
* 注意这里必须dashboad域名，如下：
```
 Common Name (eg, your name or your server's hostname) []:kubernetes-dashboard.gzky.com
```
openssl x509 -req -sha256 -days 365 -in dashboard.csr -signkey dashboard.key -out dashboard.crt

dashboard使用证书,建secret
kubectl create secret generic kubernetes-dashboard-certs --from-file=$HOME/certs -n kube-system

修改dashboard secret tsl文件
cat /root/certs/dashboard.key | base64 | tr -d '\n'
cat /root/certs/dashboard.crt | base64 | tr -d '\n'

获取密码：
kubectl -n kube-system get secret | grep kubernetes-dashboard
kubectl describe -n kube-system secret/kubernetes-dashboard-admin-token-4sbht

```
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbi10b2tlbi1yYjlnNCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImU5N2E3MjEzLTZlY2YtMTFlOC05ZTQ3LTAwMWE0YTE2MDE4NyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiJ9.dYJaYmOmmKSlu_dTaRrwcAGGeWg0s1OnPXKg6j915jkGzWuR4wwaojnQn1lA8SxJClrmfTKdH4siB_deilSFJdjcnSLGdVfDv_NLNUeC39FgecznZiNjT8VqScW_GL7GbFrWqYN1RjWR2gyNZXV8CwPeIsbfDI-u_xkXYHpEGyv1VH3s4WEYILksGD1VbQKAUiyu-IDjE2k_qwN5GD4xNt3B7zBRclExO1Tvduw-HhtiuLu-Yx-XfkrvfwXiDs5Fk6do_S3ls130_PlOW45-pRy5lr7g85tHpX6-Tw59npVcZIMw79cr02fGhQfEiLEo0rsbvBm2Cbb5MEsbDbJ6dg
```
