参考地址：
http://containerized.me/how-to-deploy-a-postgresql-cluster-on-kubernetes-openebs/
https://github.com/openebs/openebs/tree/master/k8s/demo/crunchy-postgres

可以不安装
yum install postgresql

登陆pod
```
kubectl exec -it pgset-0 -- /bin/sh
```

psql 登陆
```
psql
```

创建用户
```
CREATE USER user1 WITH PASSWORD '123';
```

创建数据库
```
GRANT ALL PRIVILEGES ON DATABASE exampledb TO user1;
```

以新用户登陆
```
psql -U user1 -W  -d test
```

创建表
```
create table Person (id int, name varchar(255), email varchar(255), primary key(id));
```

插入数据
```
insert into  person values (1,'Lili','lili@123.com');
insert into  person values (2,'TOM','tom@123.com');
```


