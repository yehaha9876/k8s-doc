1. 下载VirtualBox
https://www.virtualbox.org/wiki/Linux_Downloads

sudo yum install -y VirtualBox-5.2-5.2.12_122591_el7-1.x86_64.rpm

2. 安装kubectl

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubectl --nogpgcheck

我这里网络不可访问，用下面的试一下

curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
貌似动了
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl


检查配置
kubectl cluster-info

Enabling shell autocompletion

echo "source <(kubectl completion bash)" >> ~/.bashrc

3. 安装Minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.26.1/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

4. 运行
minikube start  --vm-driver=virtualbox

报错：
WARNING: The vboxdrv kernel module is not loaded. Either there is no module\n         available for the current kernel (3.10.0-693.el7.x86_64) or it failed to\n         load. Please recompile the kernel module and install it by\n\n           sudo /sbin/vboxconfig\n\n         You will not be able to start VMs until this problem is fixed.\n5.2.12r122591


sudo yum install kernel-devel 
sudo /usr/lib/virtualbox/vboxdrv.sh setup

wait...
[sean@bogon workspace]$ minikube start  --vm-driver=virtualbox
Starting local Kubernetes v1.10.0 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...


