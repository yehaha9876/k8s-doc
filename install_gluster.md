参考文档：
https://wiki.centos.org/SpecialInterestGroup/Storage/gluster-Quickstart
http://download.gluster.org/pub/gluster/glusterfs
https://docs.gluster.org/en/latest/Administrator%20Guide/Setting%20Up%20Volumes/#setting-up-glusterfs-volumes

## 安装：
增加源
vi /etc/yum.repos.d/glusterfs.repo
```
[glusterfs-4.1]
name=glusterfs 4.1
baseurl=https://buildlogs.centos.org/centos/7/storage/x86_64/gluster-4.1/
enabled=1
gpgcheck=0
```

yum makecache fast

在server中安装
yum install -y glusterfs glusterfs-server glusterfs-fuse glusterfs-rdma

启动服务
systemctl start glusterd.service
守护进程
systemctl enable glusterd.service

## 增加节点
所有节点
vi /etc/hosts
```
10.6.0.140 swarm-manager
10.6.0.192 swarm-node-1
```

加入节点
gluster peer probe swarm-manager
gluster peer probe swarm-node-1

查看节点状态
gluster peer status

### 注意
这里安装时需要确认，节点的ip，和host都被加入到集群中 
```
Hostname: swarm-node-1
Uuid: e09cb003-d4bf-4ad4-ab01-cfe9e08c05a7
State: Peer in Cluster (Connected)
Other names:
192.168.3.77

Hostname: swarm-node-2
Uuid: e17fc2d4-5a2e-42fc-9f7a-db3aa28cbdb0
State: Peer in Cluster (Connected)
Other names:
192.168.3.62

```

每个gluster集群增加共享目录
mkdir -p /opt/gluster/data

使用复制模式创建 volume

gluster volume create test-volume replica 2 transport tcp server1:/exp1 server2:/exp2

查看状态


启用volume
gluster volume start test-volume

## gluster 客户端
yum install -y glusterfs glusterfs-fuse
mkdir -p /opt/gfsmnt
mount -t glusterfs swarm-manager:models /opt/gfsmnt/
注意： glusterfs hosts 必须加入

查看：
df -h

## 其他

gluster 性能调优：

开启 指定 volume 的配额： (models 为 volume 名称)
  gluster volume quota models enable

  限制 models 中 / (既总目录) 最大使用 80GB 空间
  gluster volume quota models limit-usage / 80GB

#设置 cache 4GB
  gluster volume set models performance.cache-size 4GB

#开启 异步 ， 后台操作
  gluster volume set models performance.flush-behind on

#设置 io 线程 32
  gluster volume set models performance.io-thread-count 32

#设置 回写 (写数据时间，先写入缓存内，再写入硬盘)
  gluster volume set models performance.write-behind on


