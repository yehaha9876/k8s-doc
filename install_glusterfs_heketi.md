最低配置要求：
必须3太glusterfs节点，每个节点单独空分区，空间至少32G

文档地址：
https://github.com/heketi/heketi/tree/master/extras/kubernetes
https://github.com/gluster/gluster-kubernetes
https://docs.openshift.com/container-platform/3.5/install_config/storage_examples/dedicated_gluster_dynamic_example.html

安装过程：
gluster peer status
确认iphost都在，如果没有单独加一下host

### 安装程序
 yum install heketi heketi-client -y

### 生成公钥并使可以无需密码登陆
ssh-keygen -f /etc/heketi/heketi_key -t rsa -N ''
ssh-copy-id -i /etc/heketi/heketi_key.pub root@gluster23.rhs
ssh-copy-id -i /etc/heketi/heketi_key.pub root@gluster24.rhs
ssh-copy-id -i /etc/heketi/heketi_key.pub root@gluster25.rhs

### 编辑配置文件
vi /etc/heketi/heketi.json
```
"executor": "ssh", 

"_sshexec_comment": "SSH username and private key file information",
"sshexec": {
  "keyfile": "/etc/heketi/heketi_key", 
  "user": "root", 
  "port": "22", 
  "fstab": "/etc/fstab" 
},
```
### 重启

```
# systemctl restart heketi
# systemctl enable heketi
```

### 测试
curl http://glusterclient2.rhs:8080/hello

### 倒入env
export HEKETI_CLI_SERVER=http://glusterclient2.rhs:8080

### 编辑glusterfs集群配置信息
```
{
  "clusters": [
    {
      "nodes": [
        {
          "node": {
            "hostnames": {
              "manage": [
                "gluster23.rhs"
              ],
              "storage": [
                "192.168.1.200"
              ]
            },
            "zone": 1
          },
          "devices": [
            "/dev/sde",
            "/dev/sdf"
          ]
        }
		]
	}
}
```

### 导入配置

```
heketi-cli topology load --json=topology.json
```

### 测试一个volume

heketi-cli volume create --size=1
gluster volume info



## Dynamically Provision a Volume
建立storige classs
```
kind: StorageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: gluster-dyn
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: "http://glusterclient2.rhs:8080" 
  restauthenabled: "false" 
```

### 创建pvc
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: gluster-dyn-pvc
 annotations:
   volume.beta.kubernetes.io/storage-class: gluster-dyn
spec:
 accessModes:
  - ReadWriteMany
 resources:
   requests:
        storage: 10Mi
```

### 确认volume
 gluster volume info

### 创建pod
```
apiVersion: v1
kind: Pod
metadata:
  name: gluster-pod1
  labels:
    name: gluster-pod1
spec:
  containers:
  - name: gluster-pod1
    image: nginx
    ports:
    - name: web
      containerPort: 80
    securityContext:
      privileged: true
    volumeMounts:
    - name: gluster-vol1
      mountPath: /usr/share/nginx/html
  volumes:
  - name: gluster-vol1
    persistentVolumeClaim:
      claimName: gluster-dyn-pvc
```

连接pod
```
# oc exec -ti gluster-pod1 /bin/sh
$ cd /usr/share/nginx/html
$ echo 'Hello World from GlusterFS!!!' > index.html
$ ls
index.html
$ exit
```
# curl http://10.38.0.0
Hello World from GlusterFS!!!



安装完成后查看glusterfs挂在卷
 gluster volume info

安装失败后可以用命令删除 volume group
See if old volume group is preset with 
```
vgdisplay.
```

Remove group with 
```
vgremove -f <vg_name>
```

无法启动，确认帐号，或者直接编辑启动文件，用root身份
```

[Unit]
Description=Heketi Server

[Service]
Type=simple
WorkingDirectory=/var/lib/heketi
EnvironmentFile=-/etc/heketi/heketi.json
ExecStart=/usr/bin/heketi --config=/etc/heketi/heketi.json
Restart=on-failure
StandardOutput=syslog
StandardError=syslog

[Install]
WantedBy=multi-user.target
```
