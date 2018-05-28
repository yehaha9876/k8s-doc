## server
安装：
yum -y install nfs-utils rpcbind

mkdir -p /data/nfs_data
chmod 666 /data/nfs_data
vim /etc/exports
```
/data/nfs_data 192.168.2.0/24(rw,no_root_squash,no_all_squash,sync)
```

更新配在:

exportfs -r

启动:
service rpcbind start
service nfs start

rpcinfo -p localhost

查看挂载:
showmount -e localhost


## client

yum -y install nfs-utils
showmount -e server_host

挂载:  
mount -t nfs 192.168.2.203:/data/lys /lys -o proto=tcp -o nolock

查看：  
df -h

卸载：
umount /path/to

参考文档：https://www.cnblogs.com/liuyisai/p/5992511.html
