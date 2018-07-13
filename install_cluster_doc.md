1. 安装keepalive
yum install -y keepalived

systemctl enable keepalived && systemctl restart keepalived

备份配置文件
mv /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf.bak

创建 keepalive config
vi /etc/keepalived/keepalived.conf

vi /etc/keepalived/check_apiserver.sh

systemctl restart keepalived

2. 修改host，添加master对应ip

2. 使用kubeadm安装master1
修改base下配置文件，为master ip，loadbancd ip
默认配置：https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/

初始化master
```
kubeadm init --config kubeadm-config.yaml
```

如果卡住多半是image没有下来，可以用查看需要的images
```
kubeadm config images list --kubernetes-version=v1.11.0
```

记录输出的下join命令
kubeadm join 192.168.3.99:6443 --token f2ae8o.bi8wt89ox01iu1co --discovery-token-ca-cert-hash sha256:b7e893aecd5b73cc96545ae7094f3d1df5d8d1a8840a59b9ffc3a2746bcdfc92

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


执行命令查看pod
```
[root@master base]# kubectl get pods --all-namespaces 
NAMESPACE     NAME                             READY     STATUS    RESTARTS   AGE
kube-system   coredns-78fcdf6894-5sqlm         0/1       Pending   0          2m
kube-system   coredns-78fcdf6894-vpd44         0/1       Pending   0          2m
kube-system   etcd-master                      1/1       Running   0          1m
kube-system   kube-apiserver-master            1/1       Running   0          1m
kube-system   kube-controller-manager-master   1/1       Running   0          2m
kube-system   kube-proxy-wl4q8                 1/1       Running   0          2m
kube-system   kube-scheduler-master            1/1       Running   0          2m
```
```
[root@master base]# kubectl get nodes
NAME      STATUS     ROLES     AGE       VERSION
master    NotReady   master    4m        v1.11.0
```

注意这里NotReady是因为没有网络插件

## 安装calico网络插件
修改配置文件正的etcd endpoionts
注意： 如果第一遍没有创建成功网络，coredns就可能失败再也起不来,必须重新kubeadm init

修改secret.yaml
替换其中的 tsl 配置
vi secret.yaml 
cat /etc/kubernetes/pki/etcd/ca.crt | base64 | tr -d '\n'
cat /etc/kubernetes/pki/apiserver-etcd-client.key | base64 | tr -d '\n'
cat /etc/kubernetes/pki/apiserver-etcd-client.crt | base64 | tr -d '\n'

```
kubectl apply -f secret.yaml 
kubectl apply -f rbac.yaml 
kubectl apply -f calico.yaml
```

这时候基本组建已经安装成功

先拉去master需要的镜像

先复制etcd配置，和tsl证书
```
USER=root # customizable
CONTROL_PLANE_IPS="192.168.3.63 192.168.3.77"
for host in ${CONTROL_PLANE_IPS}; do
    scp /etc/kubernetes/pki/ca.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/ca.key "${USER}"@$host:
    scp /etc/kubernetes/pki/sa.key "${USER}"@$host:
    scp /etc/kubernetes/pki/sa.pub "${USER}"@$host:
    scp /etc/kubernetes/pki/front-proxy-ca.crt "${USER}"@$host:
    scp /etc/kubernetes/pki/front-proxy-ca.key "${USER}"@$host:
    scp /etc/kubernetes/pki/etcd/ca.crt "${USER}"@$host:etcd-ca.crt
    scp /etc/kubernetes/pki/etcd/ca.key "${USER}"@$host:etcd-ca.key
    scp /etc/kubernetes/admin.conf "${USER}"@$host:
done
```

### 安装第二台master
mkdir -p /etc/kubernetes/pki/etcd
mv ca.crt /etc/kubernetes/pki/
mv ca.key /etc/kubernetes/pki/
mv sa.pub /etc/kubernetes/pki/
mv sa.key /etc/kubernetes/pki/
mv front-proxy-ca.crt /etc/kubernetes/pki/
mv front-proxy-ca.key /etc/kubernetes/pki/
mv etcd-ca.crt /etc/kubernetes/pki/etcd/ca.crt
mv etcd-ca.key /etc/kubernetes/pki/etcd/ca.key
mv admin.conf /etc/kubernetes/admin.conf

从master拷贝，修改kubeadm-config.yaml
注意项：
initial-cluster: "master=https://192.168.3.60:2380,master2=https://192.168.3.77:2380"
initial-cluster-state: existing
name： master2

使用kubeadm配置kubelet
kubeadm alpha phase certs all --config kubeadm-config.yaml
kubeadm alpha phase kubelet config write-to-disk --config kubeadm-config.yaml
kubeadm alpha phase kubelet write-env-file --config kubeadm-config.yaml
kubeadm alpha phase kubeconfig kubelet --config kubeadm-config.yaml
systemctl start kubelet

CP0_IP=192.168.3.63
CP0_HOSTNAME=master2
CP1_IP=192.168.3.60
CP1_HOSTNAME=master

使用etcd命令增加member
```
CP0_IP=192.168.3.60
CP0_HOSTNAME=master
CP1_IP=192.168.3.63
CP1_HOSTNAME=master2
KUBECONFIG=/etc/kubernetes/admin.conf kubectl exec -n kube-system etcd-${CP0_HOSTNAME} -- etcdctl --ca-file /etc/kubernetes/pki/etcd/ca.crt --cert-file /etc/kubernetes/pki/etcd/peer.crt --key-file /etc/kubernetes/pki/etcd/peer.key --endpoints=https://${CP0_IP}:2379 member add ${CP1_HOSTNAME} https://${CP1_IP}:2380
```
创建etcd
kubeadm alpha phase etcd local --config kubeadm-config.yaml

这时候第一台etcd会挂掉，没关系只要第二太加进来就能启动成功了

查看一下
/etc/kubernetes/manifests/etcd.yaml
--name=master2 是不master2

等一会儿执行下面命令，会发现etcd机器变成两台
 KUBECONFIG=/etc/kubernetes/admin.conf kubectl exec -n kube-system etcd-${CP0_HOSTNAME} -- etcdctl --ca-file /etc/kubernetes/pki/etcd/ca.crt --cert-file /etc/kubernetes/pki/etcd/peer.crt --key-file /etc/kubernetes/pki/etcd/peer.key --endpoints=https://${CP0_IP}:2379 member list


然后创建master需要的pod
kubeadm alpha phase kubeconfig all --config kubeadm-config.yaml
kubeadm alpha phase controlplane all --config kubeadm-config.yaml
kubeadm alpha phase mark-master --config kubeadm-config.yaml


### 加入第三台机器
注意先下载镜像

从master拷贝，修改kubeadm-config.yaml
注意项：
initial-cluster: "master=https://192.168.3.60:2380,master3=https://192.168.3.77:2380,master2=https://192.168.3.77:2380"
initial-cluster-state: existing
name： master3

使用kubeadm配置kubelet
kubeadm alpha phase certs all --config kubeadm-config.yaml
kubeadm alpha phase kubelet config write-to-disk --config kubeadm-config.yaml
kubeadm alpha phase kubelet write-env-file --config kubeadm-config.yaml
kubeadm alpha phase kubeconfig kubelet --config kubeadm-config.yaml
systemctl start kubelet

在原etcd上增加etcd
```
CP0_IP=192.168.3.60
CP0_HOSTNAME=master
CP2_IP=192.168.3.77
CP2_HOSTNAME=master3
KUBECONFIG=/etc/kubernetes/admin.conf kubectl exec -n kube-system etcd-${CP0_HOSTNAME} -- etcdctl --ca-file /etc/kubernetes/pki/etcd/ca.crt --cert-file /etc/kubernetes/pki/etcd/peer.crt --key-file /etc/kubernetes/pki/etcd/peer.key --endpoints=https://${CP0_IP}:2379 member add ${CP2_HOSTNAME} https://${CP2_IP}:2380
```
创建etcd
kubeadm alpha phase etcd local --config kubeadm-config.yaml


创建master需要的pod
kubeadm alpha phase kubeconfig all --config kubeadm-config.yaml
kubeadm alpha phase controlplane all --config kubeadm-config.yaml
kubeadm alpha phase mark-master --config kubeadm-config.yaml

然后确认安装
kubectl get pods --all-namespaces
kubectl get nodes

文档地址：
https://kubernetes.io/docs/tasks/tools/install-kubeadm/#be
https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm
https://kubernetes.io/docs/setup/independent/high-availability/
https://kubernetes.io/docs/tasks/administer-cluster/setup-ha-etcd-with-kubeadm/

