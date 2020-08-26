### rabbitmq

- 部署：

阿里云创建NAS共享存储的StorageClass：

```bash
kubectl apply -f rabbitmq-sc.yaml
```

```bash
kubectl apply -f ./
```

部署完毕后，

```bash
kubectl get all -n public-service

NAME                READY   STATUS    RESTARTS   AGE
pod/rmq-cluster-0   1/1     Running   0          4h
pod/rmq-cluster-1   1/1     Running   0          4h
pod/rmq-cluster-2   1/1     Running   0          4h

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)              AGE
service/rmq-cluster   ClusterIP   172.21.11.245   <none>        15672/TCP,5672/TCP   4h

NAME                           READY   AGE
statefulset.apps/rmq-cluster   3/3     4h
```

添加hosts：`rabbitmq.lzxlinux.com`，使用初始账号密码`guest/guest`登录即可。

---
