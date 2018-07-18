在每个master节点上安装
yum install -y keepalived

vi /etc/keepalived/keepalived.conf
```
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 2
    weight -5
    fall 3
    rise 2
}
vrrp_instance VI_1 {
    state MASTER
    interface nm-bond
    mcast_src_ip 192.168.3.60
    virtual_router_id 51
    priority 102
    advert_int 2
    authentication {
            auth_type PASS
            auth_pass 4cdf7dc3b4c90194d1600c483e10ad1d
        }
    virtual_ipaddress {
            192.168.3.99
        }
    track_script {
           chk_apiserver
        }
}
```

service keepalived start

配置nginx
```
stream {
    upstream apiserver {
            server 192.168.3.60:6443 weight=5 max_fails=3 fail_timeout=30s;
            server 192.168.3.63:6443 weight=5 max_fails=3 fail_timeout=30s;
            server 192.168.3.77:6443 weight=5 max_fails=3 fail_timeout=30s;
        }

    server {
            listen 16443;
            proxy_connect_timeout 1s;
            proxy_timeout 3s;
            proxy_pass apiserver;
        }
}
```
可以修改hostname


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

## 这里需要先创建calico网络

## 复制master证书到其他master
scp -r /etc/kubernetes/pki master2:/etc/kubernetes/
scp -r /etc/kubernetes/pki master3:/etc/kubernetes/

## 其他master节点运行
kubeadm init --config=kubeadm-init.yaml
如果正常运行，节点会生相同的join token 和 sha256 cert hash


把当前所有的节点设置为master
kubectl taint nodes --all node-role.kubernetes.io/master-

## 修改master节点的kube-apiserver.yaml
vi /etc/kubernetes/manifests/kube-apiserver.yaml
增加参数,这样service kubernetes的endpoint就不会只有一个而且一直在变
```
- --apiserver-count=3
- --endpoint-reconciler-type=lease
```

## 设置所有node节点使用负载均衡
$ sed -i "s/192.168.20.29:6443/192.168.20.10:16443/g" /etc/kubernetes/bootstrap-kubelet.conf
$ sed -i "s/192.168.20.27:6443/192.168.20.10:16443/g" /etc/kubernetes/kubelet.conf

systemctl restart docker kubelet

参考文档
https://github.com/cookeem/kubeadm-ha/blob/master/README_CN.md


安装 canal (calico/flannel)
参考文档
https://github.com/projectcalico/canal
https://docs.projectcalico.org/v3.1/getting-started/kubernetes/

安装dashboard
参考文档
https://github.com/kubernetes/dashboard
