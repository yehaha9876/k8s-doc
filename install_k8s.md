sudo yum install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm
sudo yum reinstall -y docker-ce-17.03.0.ce

启动docker
docker -v
sudo systemctl start docker

## Master
设置Cgroup driver
sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

初始化k8s
kubeadm init   --kubernetes-version=v1.10.3   --pod-network-cidr=10.244.0.0/16   --apiserver-advertise-address=192.168.3.60 --ignore-preflight-errors=Swap > install.log 2>&1

获取状态
kubectl get cs

kubectl get pods --all-namespaces -o wide

查看不成功的某个pod
kubectl describe pod kube-dns-86f4d74b45-jljvz --namespace=kube-system


## Node节点执行

需要安装的镜像
k8s.gcr.io/pause-amd64        3.1                 f893dc52df88        6 hours ago         742 kB
k8s.gcr.io/kube-proxy-amd64   v1.10.3             cf34e1670980        7 hours ago         97.1 MB
quay.io/coreos/flannel        v0.10.0-amd64       f0fad859c909        4 months ago        44.6 MB


查看注册过程的错误：
journalctl -u kubelet -f

设置kubelet cgroup-driver 跟 docker 一致:
```
sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
systemctl daemon-reload
systemctl restart kubelet
```


错误：
```
/proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
```

vi /etc/sysctl.conf
```
net.bridge.bridge-nf-call-iptables = 1
```
sysctl -p

文档:
https://blog.csdn.net/tycoon1988/article/details/40826235
https://blog.csdn.net/dog250/article/details/8654487


关闭swap


错误：
unable to check if the container runtime at "/var/run/dockershim.sock" is running: exit status 1
解决方法:
kubeadm init --ignore-preflight-errors=cri


错误
unable to check if the container runtime at "/var/run/dockershim.sock" is running: exit status 1

```
join 加入参数 --ignore-preflight-errors cri
```
