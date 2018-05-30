参考文档：
https://www.kubernetes.org.cn/3894.html
https://github.com/kubernetes-incubator/external-storage
https://jimmysong.io/kubernetes-handbook/practice/using-nfs-for-persistent-storage.html

感觉一个鸟样子
遇到问题：

kubectl get sc 正常

kubectl get pv 正常

kubectl get pvc //pending

还最新版的ok。。。。

使用的时候注意：
```
  volumeClaimTemplates:
  - metadata:
      name: storage
    spec:
      storageClassName: managed-nfs-storage //这里需要制定storage class
      accessModes: [ ReadWriteOnce ]
      resources:
        requests:
          storage: 12Gi
```
也可以设置wei默认
kubectl patch storageclass default -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
