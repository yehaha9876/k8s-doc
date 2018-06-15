安装：

```
wget https://github.com/coreos/etcd/releases/download/v3.3.7/etcd-v3.3.7-linux-amd64.tar.gz

tar xzvf etcd-v3.3.7-linux-amd64.tar.gz

cp etcd-v3.3.7-linux-amd64/etcd* /usr/bin/
```

查看版本：
```
etcd --version
```

```
ETCDCTL_API=3
etcdctl --version

```

add to service

vi /etc/systemd/system/etcd.service
```
[Unit]
Description=etcd.service
[Service]
Type=notify
TimeoutStartSec=0
Restart=always
ExecStartPre=-/bin/mkdir -p /data/etcd
ExecStart=/usr/bin/etcd \
  --name etcd2 \
  --listen-client-urls http://0.0.0.0:2379 \
  --advertise-client-urls http://192.168.3.63:2379 \
  --data-dir /data/etcd \
  --initial-advertise-peer-urls http://192.168.3.63:2380 \
  --initial-cluster-state new \
  --initial-cluster-token 7f276c.0741d82a5337f526 \
  --listen-peer-urls http://0.0.0.0:2380 \
  --initial-cluster ectd2=http://192.168.3.63:2380
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

添加守护
```
systemctl enable /etc/systemd/system/etcd.service
```


启动
```
systemctl start etcd.service
```
```
systemctl daemon-reload
```

测试：
etcdctl put foo bar
etcdctl get foo

etcdctl --endpoints=http://192.168.3.63:2379 get foo

ETCDCTL_API=3 etcdctl --endpoints localhost:2379 snapshot save snapshot.db
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db --name m3 --data-dir=/home/etcd_data

查看节点
etcdctl member list

查看日志
 journalctl -u etcd -f


# 增加新节点
现在集群中执行：
etcdctl member add etcd3 http://192.168.3.77:2380
启动项里
vi /etc/systemd/system/etcd.service
```
[Unit]
Description=etcd.service
[Service]
Type=notify
TimeoutStartSec=0
Restart=always
ExecStartPre=-/bin/mkdir -p /data/etcd
ExecStart=/usr/bin/etcd \
  --name etcd3 \
  --listen-client-urls http://0.0.0.0:2379 \
  --advertise-client-urls http://192.168.3.63:2379 \
  --data-dir /data/etcd \
  --initial-advertise-peer-urls http://192.168.3.63:2380 \
  --initial-cluster-state existing\
  --initial-cluster-token 7f276c.0741d82a5337f526 \
  --listen-peer-urls http://0.0.0.0:2380 \
  --initial-cluster etcd1=http://192.168.3.60:2380,etcd2=http://192.168.3.63:2380,etcd3=http://192.168.3.77:2380

[Install]
WantedBy=multi-user.target
```

命令启动，可以用环境变量

```
export ETCD_NAME=etcd3
export ETCD_INITIAL_CLUSTER="etcd1=http://192.168.3.60:2380,etcd2=http://192.168.3.63:2380,etcd3=http://192.168.3.77:2380"
export ETCD_INITIAL_CLUSTER_STATE=existing
export ETCD_INITIAL_CLUSTER_TOKEN=etcd1
```
