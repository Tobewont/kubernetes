### apollo

- 数据库导入：

首先部署数据库，并导入sql

```bash
git clone https://github.com/ctripcorp/apollo.git

mysql -uroot -p123456789 < apollo/scripts/sql/apolloportaldb.sql

mysql -uroot -p123456789 < apollo/scripts/sql/apolloconfigdb.sql
```

- 部署：

> ConfigMap中注意修改mysql的ip:port及账号密码；
> ConfigMap中注意根据namespace修改url；
> ConfigMap中注意根据部署情况修改env。

```bash
kubectl apply -f public-service-ns.yaml

kubectl apply -f apollo-configservice/

kubectl apply -f apollo-adminservice/

kubectl apply -f apollo-portal/
```

```bash
kubectl get svc -n public-service

NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
apollo-adminservice    ClusterIP   10.110.239.51    <none>        8090/TCP   14s
apollo-configservice   ClusterIP   10.103.129.253   <none>        8080/TCP   14s
apollo-portal          ClusterIP   10.105.237.238   <none>        8070/TCP   13s

kubectl get pod -n public-service 

NAME                                    READY   STATUS    RESTARTS   AGE
apollo-adminservice-765497fbbc-htskm    1/1     Running   0          35s
apollo-configservice-64b4d77457-8w59g   1/1     Running   0          35s
apollo-portal-5f585dc954-th96h          1/1     Running   0          34s
```

- 访问ui：

添加hosts：`apollo.lzxlinux.com`，账号/密码：`apollo`/`admin`。

---
