在每个master节点上安装
yum install -y keepalived

# 主master初始化程序
kubeadm init --config=kubeadm-init.yaml

kubeadm join 192.168.3.60:6443 --token 7f276c.0741d82a5337f526 --discovery-token-ca-cert-hash sha256:f5a90bc04c7ee7ea1a7aac0c3d03bd17c2eb0c0f391a61d135bc5c40ddea422f  

配置kubelet
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

kubctl get pods


安装 canal (calico/flannel)
参考文档
https://github.com/projectcalico/canal
https://docs.projectcalico.org/v3.1/getting-started/kubernetes/

安装dashboard
参考文档
https://github.com/kubernetes/dashboard
