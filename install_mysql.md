参考文档：
https://kubernetes.io/docs/tasks/run-application/run-replicated-stateful-application/

创建StatefulSet的时候需要存储卷
```
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```
这里用的是利用 nfs-client-provisioner 做动态存储卷，使用的是默认的 storageClass
```
kubectl get sc
```

搭建成功后可以设置成默认sc
kubectl patch storageclass nameOfSCClass -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}

mysql image:
```
/home/sean/workspace/examples/staging/storage/mysql-galera/image
```
