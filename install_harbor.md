Hardware
Resource  Capacity  Description
CPU minimal 2 CPU 4 CPU is prefered
Mem minimal 4GB 8GB is prefered
Disk  minimal 40GB  160GB is prefered

### 安装docker-compose
yum install docker-compose

docker-compose --version

### 下载harbor
wget https://github.com/vmware/harbor/archive/v1.4.1.tar.gz

### 编辑harbor.cfg文件
修改hostname,ui_url_protocol,ustomize_crt,ssl_cert，ssl_cert_key，harbor_admin_password

### 安装
执行：./prepare， ./install.sh

### 停止
docker-compose down

默认用户名密码
admin/Harbor12345

### 相关文档
[安装文档](https://github.com/vmware/harbor/blob/master/docs/installation_guide.md)
[https 文档](https://github.com/vmware/harbor/blob/master/docs/configure_https.md)
