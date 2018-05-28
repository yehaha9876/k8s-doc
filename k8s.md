相关文档
https://www.kubernetes.org.cn/doc-6
https://kubernetes.io/docs/setup/building-from-source/

http://www.do1618.com/archives/1164/
http://blog.51cto.com/devingeng/2096495

另一种解决慢的方法
https://mritd.me/2016/10/29/set-up-kubernetes-cluster-by-kubeadm/

代理镜像
https://my.oschina.net/u/2306127/blog/1628082

文档
https://jimmysong.io/kubernetes-handbook

https://kubernetes.io/docs/home/?path=users&persona=app-developer&level=foundational

yum安装慢可以加上
--nogpgcheck 参数

1. 安装虚拟机
yum install vagrant
可以用rpm包
https://www.vagrantup.com/downloads.html

vagrant plugin install vagrant-libvirt
遇到问题：
  timed out (https://gems.hashicorp.com/specs.4.8.gz)
解决：
vi /opt/vagrant/embedded/gems/2.1.1/gems/vagrant-2.1.1/lib/vagrant/bundler.rb
修改gems.hashicorp.com 为 ruby.taobao.org

报错：
issues. The error from Bundler is:

ERROR: Failed to build gem native extension.

    current directory: /home/sean/.vagrant.d/gems/2.4.4/gems/nokogiri-1.8.2/ext/nokogiri
/opt/vagrant/embedded/bin/ruby -r ./siteconf20180514-15754-1oycie.rb extconf.rb
checking if the C compiler accepts ... *** extconf.rb failed ***
Could not create Makefile due to some reason, probably lack of necessary
libraries and/or headers.  Check the mkmf.log file for more details.  You may
need configuration options.

查看日志没有gcc

解决：
yum install gcc

遇到问题：
ERROR: Failed to build gem native extension.

    current directory: /home/sean/.vagrant.d/gems/2.4.4/gems/ruby-libvirt-0.7.0/ext/libvirt
/opt/vagrant/embedded/bin/ruby -r ./siteconf20180514-16849-139ekum.rb extconf.rb
*** extconf.rb failed ***

解决：
yum install libvirt libvirt-devel


2. 安装k8s

sudo yum install -y yum-utils device-mapper-persistent-data lvm2


sudo yum -y install golang

git clone git@github.com:kubernetes/kubernetes.git kubernetes

问题：
+++ [0514 18:24:18] Verifying Prerequisites....
Can't connect to 'docker' daemon.  please fix and retry.

解决：
sudo groupadd docker
sudo usermod -aG docker $(whoami)
sudo service docker start


问题：
[sean@bogon kubernetes1]$ make quick-release
+++ [0515 10:04:20] Verifying Prerequisites....
+++ [0515 10:04:20] Building Docker image kube-build:build-3d01223430-5-v1.10.2-1
+++ Docker build command failed for kube-build:build-3d01223430-5-v1.10.2-1

Sending build context to Docker daemon 8.704 kB
Step 1/16 : FROM k8s.gcr.io/kube-cross:v1.10.2-1
Trying to pull repository k8s.gcr.io/kube-cross ... 
Get https://k8s.gcr.io/v1/_ping: dial tcp 74.125.204.82:443: i/o timeout

To retry manually, run:

解决，注册
阿里云，打开https://cr.console.aliyun.com/%20Docker#/accelerator，设置mirror
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://o5k8f1d0.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo service docker restart

centos设置mirror不成功参考
https://www.aliyun.com/product/acr?spm=a2c4g.11186623.5.3.z4Qh1m

docker info 查看是否成功

docker login registry.cn-hangzhou.aliyuncs.com
docker pull registry.cn-hangzhou.aliyuncs.com/google-containers/debian-iptables-amd64:v10
docker tag 99e59f495ffa gcr.io/google-containers/pause-amd64:3.0
docker image ls
这样就可以使用本地镜像了

k8s
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
 
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
 
EOF
 
yum -y install epel-release
 
yum clean all
 
yum makecache
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
 
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
 
EOF
 
yum -y install epel-release
 
yum clean all
 
yum makecache


2.在各节点安装kubeadm和相关工具包

1
yum -y install docker kubelet kubeadm kubectl kubernetes-cni


3.启动Docker与kubelet服务

systemctl enable docker && systemctl start docker
 
systemctl enable kubelet && systemctl start kubelet

kubectl 补全
 $ kubectl completion bash > ~/.kube/completion.bash.inc
 $ cat ~/.bash_profile
 .....
 source ~/.kube/completion.bash.inc
 .....

 $ source ~/.bash_profile

-bash: _get_comp_words_by_ref: 未找到命令

解决：
```
yum install -y bash-completion
cat >> /etc/profile
source <(kubectl completion bash)
(ctrl+D)
source /etc/profile
```

# init k8s
kubeadm init   --kubernetes-version=v1.10.2   --pod-network-cidr=10.244.0.0/16   --apiserver-advertise-address=192.168.2.92 --ignore-preflight-errors=Swap > install.log 2>&1

可以reset
kubeadm reset


```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
获取集群状态：

```
 kubectl get cs
 kubectl get pods --all-namespaces -o wide
 这里dns有一个失败的
```


在node2里执行：
kubeadm join 192.168.2.92:6443 --token p2lkmm.gge56ok8cbnif0r7 --discovery-token-ca-cert-hash sha256:857001024a61056d473c155493352ee84805b1331ade65225cbf77e72a0354b8

按装kubernetes-dashboard
$ wget https://raw.githubusercontent.com/kubernetes/dashboard/v1.8.1/src/deploy/recommended/kubernetes-dashboard.yaml

然后修改image，改成aliyu的

image: registry.cn-hangzhou.aliyuncs.com/kube_containers/kubernetes-dashboard-amd64:v1.8.3
最后以行增加
spec:
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  type: NodePort //增加这行

运行kubectl create -f kubernetes-dashboard.yaml

因为当前权限底，创建一个admin权限
vi kubernetes-dashboard-admin.rbac.yaml

增加下面代码
```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-admin
  namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard-admin
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard-admin
  namespace: kube-system
```

运行命令
kubectl create -f kubernetes-dashboard-admin.yaml

如果有问题可以删除
kubectl delete -f kubernetes-dashboard-admin.yaml

然后查看pod状态
kubectl get pods --all-namespaces

查看server状态
kubectl get service -n kube-system

``
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP   4h
kubernetes-dashboard   NodePort    10.110.101.75   <none>        443:30241/TCP   7m
```
其中31290是对外的端口
打开https://192.168.2.92:30241

令牌：
kubectl -n kube-system get secret | grep kubernetes-dashboard
```
ubernetes-dashboard-certs                       Opaque                                0         11m
kubernetes-dashboard-key-holder                  Opaque                                2         17m
kubernetes-dashboard-token-zt8cf                 kubernetes.io/service-account-token   3         11m
```

输入这个：
kubectl describe -n kube-system secret/kubernetes-dashboard-admin-token-4sbht
看到：
```
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbi10b2tlbi00c2JodCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjI3ZTExYmJhLTVhYjgtMTFlOC1iYWIxLTA4MDAyNzliOTM3MSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiJ9.v9CMblk3cTZ8xSfNrVblstj1J-4gEot7et3QMoFm6_cqRIX7qxC_xfQR4VV5CTh_55aiQJdHAUQygzShtR6ZTa96aRfnG_shLd8l3revYoDlI1FA1gJsP6I_nTdTgGH75xxg44sHzHXnmB_dhZGHsjgJGyEZFZAABgOLMlSfHei6YVpB1NtBNBZxo7NZ2PUu1nWIpiqKNSgc6z8FxN_ZYzgQkZkSSsI7mKUHL03cfNDSAFk8jElEAGnWMMQxCFUJpLHRc4OhVxr02u-NHjWI_RCJ8hJzT9iFoB3QozSnTOiD77WFtKKNefnn02rlRd14nmvUh9pv1cM-NNLXKwQ0Bw
```
```
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbi10b2tlbi03N3ZyZyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjBiZDA0ODJhLTVmZjEtMTFlOC04OWM3LTAwMWE0YTE2MDE4NyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTprdWJlcm5ldGVzLWRhc2hib2FyZC1hZG1pbiJ9.m75KI3CSB_GG_UvZ1ouQ4KnKYXum19lPeBoAgzuJ1vOrQKcjz-lhKrGsxO5wcndqpZigLSirJw0qxQTUrtQa8YIex0WwocyyNGVhc_n0W7SIdT1tFiHFzBdZ3ArdVCpfcNRjpSyo50ZfHZBO8sm0w9QUToZjQ1Wv6BEqzEere3RHaPO3dzIkx-lP8LGCAkMfkTYg2AAPp2C8WX54TeN2o6n6nQP0lzczJ07Cf7w1DBgBo8W6NiSlRB-lEcMFJSXpXKBD9ZW-DogJGfb22u7geLo2fBsxEztuSg1bMDDIcWmAqR_HTuOCaAyu2Rzrm5vUvX-xG3HwsEzNEefkiaDnjA
```


查看log
kubectl logs -f kubernetes-dashboard-69c5575d8-bchd7 --namespace=kube-system

node2 重启
master:
kubectl delete node {nodename}

node:
kubectl reset
service kubelet restart

https://192.168.2.92:30241/#!/node?namespace=default
http://192.168.2.92:32169/dashboard/db/pods?orgId=1&from=1526964505971&to=1526968105971
