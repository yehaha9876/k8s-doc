kubectl delete node n

systemctl stop kubelet && systemctl stop docker
rm -rf /var/lib/cni/ && rm -rf /var/lib/kubelet/* && rm -rf /etc/cni/

# 删除遗留的网络接口
ip a | grep -E 'docker|flannel|cni'
ip link del docker0
ip link del flannel.1
ip link del cni0

systemctl restart docker && systemctl restart kubelet
ip a | grep -E 'docker|flannel|cni'
