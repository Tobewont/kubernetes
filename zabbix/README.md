### zabbix

- 部署nfs：

选择一节点，

```bash
mkdir -p /data/mysql

yum -y install nfs-utils rpcbind

echo '/data/mysql 192.168.30.0/24(rw,sync,no_root_squash)' > /etc/exports

chmod -R 755 /data/mysql

exportfs -arv

systemctl enable rpcbind && systemctl start rpcbind

systemctl enable nfs && systemctl start nfs
```

nfs部署完毕。对于需要使用nfs的node节点，都要安装nfs：

```bash
yum -y install nfs-utils
```

- 部署：

```bash
kubectl apply -f namespace.yaml

kubectl apply -f nfs-mysql-pv.yaml

kubectl apply -f mysql/

kubectl apply -f zabbix-server/

kubectl apply -f zabbix-web/
```

```bash
kubectl get all -n monitoring

NAME                                READY   STATUS    RESTARTS   AGE
pod/mysql-0                         1/1     Running   0          5m13s
pod/zabbix-server-98fdf455b-kjfpg   2/2     Running   0          4m2s
pod/zabbix-web-7c5485fcb9-mxhsm     1/1     Running   0          104s

NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)           AGE
service/mysql           ClusterIP   None             <none>        3306/TCP          5m13s
service/zabbix-server   NodePort    10.109.117.167   <none>        10051:30051/TCP   4m2s
service/zabbix-web      ClusterIP   10.106.252.238   <none>        8080/TCP          104s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/zabbix-server   1/1     1            1           4m2s
deployment.apps/zabbix-web      1/1     1            1           104s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/zabbix-server-98fdf455b   1         1         1       4m2s
replicaset.apps/zabbix-web-7c5485fcb9     1         1         1       104s

NAME                     READY   AGE
statefulset.apps/mysql   1/1     5m13s
```

任选一个node ip，在本地添加hosts：

```a
192.168.30.129 zabbix.lzxlinux.cn
```

访问 `zabbix.lzxlinux.cn`，账号/密码：`Admin`/`zabbix`。

![Image text](https://github.com/Tobewont/kubernetes/blob/master/zabbix/img/zabbix-1.png)

Zabbix server 信息，

![Image text](https://github.com/Tobewont/kubernetes/blob/master/zabbix/img/zabbix-2.png)

- zabbix-agent：

zabbix agent 节点启动docker，

```bash
mkdir -p /data/zabbix-agent && chmod -R 755 /data/zabbix-agent

cat > /data/zabbix-agent/zabbix_agentd.conf << EOF
LogType=console
Server=192.168.30.128               #k8s master ip，部署时删除注释
StartAgents=3
ServerActive=192.168.30.128:30051
Hostname=192.168.30.129             #k8s node ip，部署时删除注释
User=zabbix
UnsafeUserParameters=1
LoadModulePath=/var/lib/zabbix/modules/
EOF

docker run -d --name zabbix-agent \
    -v /data/zabbix-agent:/etc/zabbix \
    -p 10050:10050 zabbix/zabbix-agent:latest
```

查看日志报错：`no active checks on server [192.168.30.128:30051]: host [192.168.30.129] not found`

解决：zabbix web 界面添加host，hostname 为 192.168.30.129。

Zabbix agent 信息，

![Image text](https://github.com/Tobewont/kubernetes/blob/master/zabbix/img/zabbix-3.png)

![Image text](https://github.com/Tobewont/kubernetes/blob/master/zabbix/img/zabbix-4.png)

---
