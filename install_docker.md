# 按装Docker

k8s 支持的docker版本参考：  
https://kubernetes.io/docs/tasks/tools/install-kubeadm/#installing-docker

目前k8s 版本1.10推荐使用的docker是17.03，更高版本的没有经过测试 


## 安装过程：

增加阿里云源:
```
sudo yum-config-manager  --add-repo  https://download.docker.com/linux/centos/docker-ce.repo
//或者
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

更新源:
```
sudo yum makecache fast
```

查看可用版本：
```
yum list docker-ce.x86_64 --showduplicates | sort -r
```

卸载老版本的docker：
```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

按装制定版本:
```
sudo yum install docker-ce-<VERSION STRING>
```
参考文档：
https://docs.docker.com/install/linux/docker-ce/centos/#install-docker-ce-1
https://yq.aliyun.com/articles/110806?commentId=11066

报错：
```
Error: Package: docker-ce-17.03.2.ce-1.el7.centos.x86_64 (docker-ce-stable)

           Requires: docker-ce-selinux >= 17.03.2.ce-1.el7.centos

           Available: docker-ce-selinux-17.03.0.ce-1.el7.centos.noarch (docker-ce-stable)

               docker-ce-selinux = 17.03.0.ce-1.el7.centos

           Available: docker-ce-selinux-17.03.1.ce-1.el7.centos.noarch (docker-ce-stable)

               docker-ce-selinux = 17.03.1.ce-1.el7.centos

           Available: docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch (docker-ce-stable)

               docker-ce-selinux = 17.03.2.ce-1.el7.centos

 You could try using --skip-broken to work around the problem

 You could try running: rpm -Va --nofiles --nodigest
```

先安装 docker-ce-selinux
```
sudo yum install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm
```

查看Docker版本:
```
[sean@bogon yum.repos.d]$ docker -v
Docker version 17.03.2-ce, build f5ec1e2
```

启动docker加入开机自启动
```
sudo systemctl start docker
sudo systemctl enable docker

```

报错：
```
Error starting daemon: error initializing graphdriver: driver not supported
```

到/var/log/message下查看发现

```
centos7 prior storage driver overlay2 failed: driver not supported
```

参考官方文档：
https://docs.docker.com/storage/storagedriver/overlayfs-driver/
```
Note: If you use OverlayFS, use the overlay2 driver rather than the overlay driver, because it is more efficient in terms of inode utilization. To use the new driver, you need version 4.0 or higher of the Linux kernel, unless you are a Docker EE user on RHEL or CentOS, in which case you need version 3.10.0-514 or higher of the kernel and to follow some extra steps.
```

查看我的内核
```
uname -a 
```

解决：
vim /etc/docker/daemon.json
```
{
  "storage-driver": "overlay2"
}
```

启动：
sudo systemctl start docker

查看是否设置成功

```
sudo docker info | grep "Storage Driver"
```


## 设置代理
申请代理加速地址：
https://yq.aliyun.com/articles/110806?spm=5176.8351553.0.0.23be1991oMwhXx

增加代理
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://o5k8f1d0.mirror.aliyuncs.com"]
}


Centos7可能不好用
vim /etc/sysconfig/docker

OPTIONS='--selinux-enabled --log-driver=journald --registry-mirror=http://xxxx.mirror.aliyuncs.com'
registry-mirror 输入你的镜像地址

参考文档：
[申请加速](https://yq.aliyun.com/articles/110806?spm=5176.8351553.0.0.23be1991oMwhXx)
[设置mirror](https://www.aliyun.com/product/acr?spm=a2c4g.11186623.5.3.z4Qh1m)






