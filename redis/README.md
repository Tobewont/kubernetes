### redis

- 部署nfs：

```bash
kubectl get node

NAME      STATUS   ROLES    AGE   VERSION
master1   Ready    master   33d   v1.18.6
master2   Ready    master   33d   v1.18.6
node1     Ready    <none>   33d   v1.18.6
node2     Ready    <none>   33d   v1.18.6
node3     Ready    <none>   33d   v1.18.6
```

在master2节点上做nfs共享，

```bash
yum -y install nfs-utils rpcbind

mkdir -p /data/redis/{cluster0,cluster1,cluster2,cluster3,cluster4,cluster5}

vim /etc/exports
```

```a
/data/redis/cluster0 192.168.30.0/24(rw,sync,no_root_squash)
/data/redis/cluster1 192.168.30.0/24(rw,sync,no_root_squash)
/data/redis/cluster2 192.168.30.0/24(rw,sync,no_root_squash)
/data/redis/cluster3 192.168.30.0/24(rw,sync,no_root_squash)
/data/redis/cluster4 192.168.30.0/24(rw,sync,no_root_squash)
/data/redis/cluster5 192.168.30.0/24(rw,sync,no_root_squash)
```

```bash
chmod -R 755 /data/redis

exportfs -arv

systemctl enable rpcbind && systemctl start rpcbind

systemctl enable nfs && systemctl start nfs
```

nfs部署完毕。对于需要使用nfs的node节点，都要安装nfs：

```bash
yum -y install nfs-utils
```

- 创建pv：

```bash
kubectl apply -f redis-pv.yaml

kubectl get pv

NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
nfs-pv0   1Gi        RWX            Retain           Available                                   3s
nfs-pv1   1Gi        RWX            Retain           Available                                   3s
nfs-pv2   1Gi        RWX            Retain           Available                                   3s
nfs-pv3   1Gi        RWX            Retain           Available                                   3s
nfs-pv4   1Gi        RWX            Retain           Available                                   3s
nfs-pv5   1Gi        RWX            Retain           Available                                   3s
```

- 部署：

```bash
kubectl apply -f public-service-ns.yaml

kubectl create configmap redis-conf --from-file=redis.conf -n public-service

kubectl apply -f ./

kubectl get svc -n public-service

NAME           TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
redis          ClusterIP   None          <none>        6379/TCP   20s
redis-access   ClusterIP   10.96.2.100   <none>        6379/TCP   20s

kubectl get pod -n public-service

NAME      READY   STATUS    RESTARTS   AGE
redis-0   1/1     Running   0          2m43s
redis-1   1/1     Running   0          2m18s
redis-2   1/1     Running   0          108s
redis-3   1/1     Running   0          80s
redis-4   1/1     Running   0          48s
redis-5   1/1     Running   0          30s

kubectl get pvc -n public-service

NAME           STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-redis-0   Bound    nfs-pv4   1Gi        RWX                           3m4s
data-redis-1   Bound    nfs-pv0   1Gi        RWX                           2m39s
data-redis-2   Bound    nfs-pv1   1Gi        RWX                           2m9s
data-redis-3   Bound    nfs-pv2   1Gi        RWX                           101s
data-redis-4   Bound    nfs-pv3   1Gi        RWX                           69s
data-redis-5   Bound    nfs-pv5   1Gi        RWX                           51s
```

`redis-access`这个service方便集群内访问redis集群，redis部署完毕。

---

使用Redis-tribe工具进行集群的初始化。

- 下载redis-tribe：

```bash
kubectl run -it ubuntu --image=ubuntu --restart=Never -n public-service -- bash

root@ubuntu:/# cat > /etc/apt/sources.list << EOF
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security multiverse
EOF
```

```bash
root@ubuntu:/# apt-get update

root@ubuntu:/# apt-get install -y libncursesw5 libreadline6 libtinfo5 --allow-remove-essential

root@ubuntu:/# apt-get install -y libpython2.7-stdlib python2.7 python-pip redis-tools dnsutils

root@ubuntu:/# pip install --upgrade pip

root@ubuntu:/# pip install redis-trib==0.5.1
```

- 初始化集群：

```bash
root@ubuntu:/# redis-trib.py create \
  `dig +short redis-0.redis.public-service.svc.cluster.local`:6379 \
  `dig +short redis-1.redis.public-service.svc.cluster.local`:6379 \
  `dig +short redis-2.redis.public-service.svc.cluster.local`:6379
  
root@ubuntu:/# redis-trib.py replicate \
  --master-addr `dig +short redis-0.redis.public-service.svc.cluster.local`:6379 \
  --slave-addr `dig +short redis-3.redis.public-service.svc.cluster.local`:6379

root@ubuntu:/# redis-trib.py replicate \
  --master-addr `dig +short redis-1.redis.public-service.svc.cluster.local`:6379 \
  --slave-addr `dig +short redis-4.redis.public-service.svc.cluster.local`:6379

root@ubuntu:/# redis-trib.py replicate \
  --master-addr `dig +short redis-2.redis.public-service.svc.cluster.local`:6379 \
  --slave-addr `dig +short redis-5.redis.public-service.svc.cluster.local`:6379
  
root@ubuntu:/# exit
```

- 查看集群：

```bash
kubectl exec -it redis-0 -n public-service -- bash

root@redis-0:/data# redis-cli -c

127.0.0.1:6379> CLUSTER NODES               #列出节点信息

aac2b3d320da67eedf3512ed0e38a1cdce5bc8fe 172.10.2.55:6379@16379 slave 7c4d60cf32685484ea6c5cb4493a937dfbf6b8a5 0 1592276224727 3 connected
2efad514b2f3c7fe4530dd6dc63c0df8ffdb793d 172.10.2.54:6379@16379 master - 0 1592276224224 1 connected 0-5461
524f03526a4b683d7d4de19296431810bfdc22cf 172.10.3.60:6379@16379 slave df5bc3c2e2851d63cdb9f762efde6e1b0d38efed 0 1592276223117 5 connected
7c4d60cf32685484ea6c5cb4493a937dfbf6b8a5 172.10.4.77:6379@16379 myself,master - 0 1592276224000 2 connected 5462-10922
df5bc3c2e2851d63cdb9f762efde6e1b0d38efed 172.10.3.59:6379@16379 master - 0 1592276223217 0 connected 10923-16383
c1dbaaef4a583e372c43eed52c22cd9ad7184d18 172.10.4.78:6379@16379 slave 2efad514b2f3c7fe4530dd6dc63c0df8ffdb793d 0 1592276223719 4 connected

127.0.0.1:6379> CLUSTER INFO                #集群状态

cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:5
cluster_my_epoch:2
cluster_stats_messages_ping_sent:514
cluster_stats_messages_pong_sent:491
cluster_stats_messages_meet_sent:2
cluster_stats_messages_sent:1007
cluster_stats_messages_ping_received:491
cluster_stats_messages_pong_received:516
cluster_stats_messages_received:1007
```

redis集群初始化完成，已经形成3主3从的cluster集群。

- 写入数据：

```bash
127.0.0.1:6379> SET key1 aaa
OK

127.0.0.1:6379> SET key2 bbb
-> Redirected to slot [4998] located at 172.10.2.54:6379
OK

172.10.2.54:6379> SET key3 ccc
OK
```

```bash
kubectl exec -it redis-4 -n public-service -- bash

root@redis-4:/data# redis-cli -c

127.0.0.1:6379> GET key1
-> Redirected to slot [9189] located at 172.10.4.77:6379
"aaa"

172.10.4.77:6379> GET key2
-> Redirected to slot [4998] located at 172.10.2.54:6379
"bbb"

172.10.2.54:6379> GET key3
"ccc"
```

可以看出redis cluster集群是去中心化的，每个节点都是平等的，连接哪个节点都可以获取和设置数据。

- 主从切换：

选择`redis-2`模拟宕掉，测试主从切换，

```bash
kubectl get pod -n public-service -o wide

NAME      READY   STATUS      RESTARTS   AGE   IP            NODE    NOMINATED NODE   READINESS GATES
redis-0   1/1     Running     0          62m   172.10.4.77   node1   <none>           <none>
redis-1   1/1     Running     0          62m   172.10.2.54   node2   <none>           <none>
redis-2   1/1     Running     0          61m   172.10.3.59   node3   <none>           <none>
redis-3   1/1     Running     0          61m   172.10.2.55   node2   <none>           <none>
redis-4   1/1     Running     0          61m   172.10.4.78   node1   <none>           <none>
redis-5   1/1     Running     0          61m   172.10.3.60   node3   <none>           <none>
ubuntu    0/1     Completed   0          57m   172.10.2.56   node2   <none>           <none>

kubectl exec -it redis-2 -n public-service -- bash

root@redis-2:/data# redis-cli -c

127.0.0.1:6379> ROLE

1) "master"
2) (integer) 2898
3) 1) 1) "172.10.3.60"
      2) "6379"
      3) "2898"
```

可以看到，`redis-2`是master，它的slave是`172.10.3.60`，即`redis-5`。

```bash
kubectl delete pod redis-2 -n public-service                #模拟节点宕掉

kubectl get pod redis-2 -n public-service -o wide

NAME      READY   STATUS    RESTARTS   AGE   IP            NODE    NOMINATED NODE   READINESS GATES
redis-2   1/1     Running   0          38s   172.10.3.61   node3   <none>           <none>

kubectl exec -it redis-2 -n public-service -- bash

root@redis-2:/data# redis-cli -c

127.0.0.1:6379> ROLE

1) "slave"
2) "172.10.3.60"
3) (integer) 6379
4) "connected"
5) (integer) 3430
```

```bash
kubectl exec -it redis-5 -n public-service -- bash

root@redis-5:/data# redis-cli -c

127.0.0.1:6379> ROLE

1) "master"
2) (integer) 3584
3) 1) 1) "172.10.3.61"
      2) "6379"
      3) "3570"
      
127.0.0.1:6379> CLUSTER NODES

aac2b3d320da67eedf3512ed0e38a1cdce5bc8fe 172.10.2.55:6379@16379 slave 7c4d60cf32685484ea6c5cb4493a937dfbf6b8a5 0 1592278859530 2 connected
2efad514b2f3c7fe4530dd6dc63c0df8ffdb793d 172.10.2.54:6379@16379 master - 0 1592278859000 1 connected 0-5461
c1dbaaef4a583e372c43eed52c22cd9ad7184d18 172.10.4.78:6379@16379 slave 2efad514b2f3c7fe4530dd6dc63c0df8ffdb793d 0 1592278859000 1 connected
524f03526a4b683d7d4de19296431810bfdc22cf 172.10.3.60:6379@16379 myself,master - 0 1592278857000 6 connected 10923-16383
7c4d60cf32685484ea6c5cb4493a937dfbf6b8a5 172.10.4.77:6379@16379 master - 0 1592278858021 2 connected 5462-10922
df5bc3c2e2851d63cdb9f762efde6e1b0d38efed 172.10.3.61:6379@16379 slave 524f03526a4b683d7d4de19296431810bfdc22cf 0 1592278859000 6 connected
```

可以看到，`redis-2`在重启之后变为slave，而它之前的slave——`redis-5`变为master，而且是新`redis-2`的master。

集群的主从切换没有问题。k8s部署redis cluster集群完成。

---
