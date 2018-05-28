查看什么包提供了nslookup
yum provides nslookup

Network:
cd  /etc/sysconfig/network-scripts/ 
vi ifcfg-enoXX

# 配置静态IP地址
$ vim /etc/sysconfig/network-scripts/ifcfg-ens33
TYPE="Ethernet"
BOOTPROTO="static"   # 修改 dhcp -> static
IPADDR=172.16.132.10
GATEWAY=172.16.132.2
NETMASK=255.255.255.0
DNS1=172.16.132.2
DEFROUTE="yes"
PEERDNS="yes"
PEERROUTES="yes"
IPV4_FAILURE_FATAL="no"
NAME="ens33"
UUID="018249e8-2e66-41ec-8974-032e1ca47244"
DEVICE="ens33"
ONBOOT="yes"


# 重启网卡生效
$ systemctl restart network

DNS:
# 设置DNS
$ vim /etc/resolv.conf

Ifconfig:
yum install net-tools

Linux 网桥模式：
yum install bridge-utils

host:
netstat -tnl

disable swap:
sudo swapoff -a
修改 /etc/fstab 文件，注释掉 SWAP 的自动挂载，使用free -m确认swap已经关闭。 > swappiness参数调整，修改/etc/sysctl.d/k8s.conf添加下面一行：

vm.swappiness=0

关闭防火墙：
systemctl status firewalld
systemctl stop firewalld
systemctl disable firewalld

# 禁用 selinux
$ setenforce 0
$ vim /etc/selinux/config
SELINUX=disabled

yum install telnet

增加证书
ssh-keygen
ssh-copy-id root@172.16.132.10

# 禁用ipv6
 cat /etc/sysctl.conf
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
重启
sysctl -p

yum安装慢可以加上
--nogpgcheck 参数

* sougou
yum install dpkg
mkdir sogou

* Chrom
yum install chrom
add source:
```
[google-chrome]
name=google-chrome
baseurl=http://dl.google.com/linux/chrome/rpm/stable/$basearch
enabled=1
gpgcheck=1
gpgkey=https://dl-ssl.google.com/linux/linux_signing_key.pub
```
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
sudo yum install ntfs-3g

run:
```
yum -y install google-chrome-stable
```


* yum update vs yum upgrade

repo version 6.1 6.7, update will modify config and update kernal

update only in version 6.1 to install new soft
