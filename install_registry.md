# docker registry:
## server:
创建证书：
openssl req -newkey rsa:2048 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt

启动服务：
docker run -d -p 443:5000 --restart=always --name registry \
  -v `pwd`/data:/var/lib/registry \
  -v `pwd`/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry:2

设置权健：
docker run --entrypoint htpasswd registry:2 -Bbn foo foo123  > auth/htpasswd
docker run -d -p 443:5000 --restart=always --name registry \
   -v `pwd`/auth:/auth \
   -e "REGISTRY_AUTH=htpasswd" \
   -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
   -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
   -v `pwd`/data:/var/lib/registry \
   -v `pwd`/certs:/certs \
   -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
   -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
   registry:2

设置hosts
192.168.3.63 registry.gzky.com

## client：
安装证书：
sudo mkdir -p /etc/docker/certs.d/registry.gzky.com
cp certs/domain.crt /etc/docker/certs.d/registry.gzky.com/

设置host
192.168.3.63 registry.gzky.com

登陆
docker login registry.gzky.com

创建成功后：
在本地环境加入json 项：
vi /etc/docker/daemon.json
```
"insecure-registries": ["192.168.3.60:5000"]
```

使用：
docker tag 224765a6bdbe 192.168.3.60:5000/openjdk
docker push 192.168.3.60:5000/openjdk

#k8s registroy:
参考文档：
https://hub.docker.com/r/solsson/kube-registry-proxy/

