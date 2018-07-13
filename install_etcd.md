如果时间不一致下面错误
```
etcd the clock difference against peer 578dd39e14600d48 is too high
```

yum install  ntpdate

ntpdate time.windows.com 

0 1 * * * root ntpate -s time.windows.com


先同步系统时间
rm -rf /etc/localtime
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime


修改打开文件句柄数
查看：使用ulimit -a 或者 ulimit -n
修改： vi /etc/security/limits.conf
```
*       soft    nproc       131050
*       hard    nproc       131050
*       soft    nofile      131050
*       hard    nofile      131050
```

修改打开系统总打开句柄数

echo  6553560 > /proc/sys/fs/file-max

reboot 重启

设置GOMAXPROCS 数量,etcd 推荐核心数的n+1,有人推荐核心数4倍
修改ETCD启动文件
Environment="GOMAXPROCS=10"


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

ETCDCTL_API=3 etcdctl --endpoints=http://192.168.3.63:2379 get foo

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

ETCD 防火墙
firewall-cmd --zone=public --add-port=2380/tcp --permanent
firewall-cmd --zone=public --add-port=2379/tcp --permanent
firewall-cmd --reload


ETCD 性能测试
在一下etcd以外的机器上，安装测试工具
go get -u github.com/rakyll/hey

下载测试脚本
wget https://github.com/coreos/etcd/blob/master/hack/benchmark/bench.sh

放到GOPATH下
修改脚本，$leader and $servers in the script
export GOMAXPROCS=20

注意测试机器环境GO环境变量GOMAXPROCS 和 文件打开数
