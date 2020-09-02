### consul

- PodDisruptionBudget：

k8s可以为每个应用程序创建 `PodDisruptionBudget` 对象（PDB）。PDB 将限制在同一时间因资源干扰导致的复制应用程序中宕机的 pod 数量。

可以通过两个参数来配置PodDisruptionBudget：

```a
MinAvailable：表示最小可用Pod数，表示应用Pod集群处于运行状态的最小Pod数量，或者是运行状态的Pod数同总Pod数的最小百分比

MaxUnavailable：表示最大不可用Pod数，表示应用Pod集群处于不可用状态的最大Pod数，或者是不可用状态的Pod数同总Pod数的最大百分比
```

需要注意的是，`MinAvailable`参数和`MaxUnavailable`参数只能同时配置一个。

- 部署：

```bash
kubectl apply -f public-service-ns.yaml

kubectl apply -f server/

kubectl get svc -n public-service

NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                   AGE
consul-dns      ClusterIP   10.110.235.63   <none>        53/TCP,53/UDP                                                             85s
consul-server   ClusterIP   None            <none>        8500/TCP,8600/TCP,8600/UDP,8301/TCP,8301/UDP,8302/TCP,8302/UDP,8300/TCP   85s
consul-ui       ClusterIP   10.98.220.223   <none>        80/TCP                                                                    85s


kubectl get pod -n public-service

NAME              READY   STATUS    RESTARTS   AGE
consul-server-0   1/1     Running   0          110s
consul-server-1   1/1     Running   0          107s
consul-server-2   1/1     Running   0          92s
```

- 查看集群状态：

```bash
kubectl exec -n public-service consul-server-0 -- consul members

Node             Address              Status  Type    Build  Protocol  DC   Segment
consul-server-0  172.10.135.17:8301   alive   server  1.8.3  2         dc1  <all>
consul-server-1  172.10.104.11:8301   alive   server  1.8.3  2         dc1  <all>
consul-server-2  172.10.166.136:8301  alive   server  1.8.3  2         dc1  <all>
```

- 访问ui：

添加hosts：`consul.lzxlinux.com`，访问`consul.lzxlinux.com/ui`。

- 加入client：

```bash
kubectl apply -f client/

kubectl get pod -n public-service

NAME              READY   STATUS    RESTARTS   AGE
consul-8wx22      1/1     Running   0          40s
consul-glmgs      1/1     Running   0          10s
consul-server-0   1/1     Running   0          30m
consul-server-1   1/1     Running   0          30m
consul-server-2   1/1     Running   0          30m
consul-vxbj7      1/1     Running   0          61s
```

```bash
kubectl exec -n public-service consul-server-0 -- consul members

Node             Address              Status  Type    Build  Protocol  DC   Segment
consul-server-0  172.10.135.17:8301   alive   server  1.8.3  2         dc1  <all>
consul-server-1  172.10.104.11:8301   alive   server  1.8.3  2         dc1  <all>
consul-server-2  172.10.166.136:8301  alive   server  1.8.3  2         dc1  <all>
consul-8wx22     172.10.166.138:8301  alive   client  1.8.3  2         dc1  <default>
consul-glmgs     172.10.135.19:8301   alive   client  1.8.3  2         dc1  <default>
consul-vxbj7     172.10.104.13:8301   alive   client  1.8.3  2         dc1  <default>
```

---
