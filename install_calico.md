Allow Access using a NetworkPolicy
增加pod访问策略

kubectl create -f - <<EOF
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: access-nginx
  namespace: policy-demo
spec:
  podSelector:
    matchLabels:
      run: nginx
  ingress:
    - from:
      - podSelector:
          matchLabels:
            run: access
EOF

测试

wget -q --timeout=5 nginx.policy-demo -O -

参考文档：
https://docs.projectcalico.org/v3.1/getting-started/kubernetes/tutorials/simple-policy
https://kubernetes.io/docs/concepts/services-networking/network-policies/


Namespace访问策略
https://docs.projectcalico.org/v3.1/getting-started/kubernetes/tutorials/stars-policy/

calicoctl 安装
curl -O -L https://github.com/projectcalico/calicoctl/releases/download/v3.1.3/calicoctl
chmod +x calicoctl

使用：
DATASTORE_TYPE=kubernetes KUBECONFIG=~/.kube/config calicoctl get nodes

or

export CALICO_DATASTORE_TYPE=kubernetes
export CALICO_KUBECONFIG=~/.kube/config
calicoctl get workloadendpoints

calio 原理：
https://focusvirtualization.blogspot.com/2017/03/protocol-stack-38-calico.html
http://ibash.cc/frontend/article/60/
https://blog.csdn.net/kevin3101/article/details/52368860
