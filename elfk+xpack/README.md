### elfk + x-pack

- 挂载aliyun nas：

```bash
mount -t nfs -o vers=4,minorversion=0,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport xxx.cn-hangzhou.nas.aliyuncs.com:/ /mnt

mkdir -p /mnt/elfk-data/{elasticsearch-0,elasticsearch-1,elasticsearch-2}
```

- 部署：

```bash
kubectl apply -f namespace.yaml

kubectl create configmap -n log elastic-certificates --from-file=elastic-certificates.p12=elasticsearch/elastic-certificates.p12

kubectl apply -f alicloud-nas-elfk-pv.yaml

kubectl apply -f elasticsearch-head/

kubectl apply -f elasticsearch/
```

```bash
kubectl exec -it -n log elasticsearch-0 -- bash

/usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive              #自定义密码

Enter password for [elastic]: elk-2021
Reenter password for [elastic]: elk-2021
Enter password for [apm_system]: elk-2021
Reenter password for [apm_system]: elk-2021
Enter password for [kibana]: elk-2021
Reenter password for [kibana]: elk-2021
Enter password for [logstash_system]: elk-2021
Reenter password for [logstash_system]: elk-2021
Enter password for [beats_system]: elk-2021
Reenter password for [beats_system]: elk-2021
Enter password for [remote_monitoring_user]: elk-2021
Reenter password for [remote_monitoring_user]: elk-2021
```

仅第一次建立集群时需要自定义密码，只要es集群的pv所对应的后端存储还在，即使后面重建es集群也无需再次自定义密码，当然想要修改es的密码除外。

```bash
kubectl apply -f kibana/

kubectl apply -f logstash/

kubectl apply -f filebeat/
```

```bash
kubectl get all -n log

NAME                            READY   STATUS    RESTARTS   AGE
pod/elasticsearch-0             1/1     Running   0          6h5m
pod/elasticsearch-1             1/1     Running   0          6h4m
pod/elasticsearch-2             1/1     Running   0          6h4m
pod/head-5c85b8d699-t9d4w       1/1     Running   0          3d8h
pod/kibana-5cfb7767fd-5x9vb     1/1     Running   0          3h45m
pod/logstash-746ddb77cc-bch5j   1/1     Running   0          25m
pod/logstash-746ddb77cc-mzl8c   1/1     Running   0          25m
pod/logstash-746ddb77cc-t9hvp   1/1     Running   0          25m

NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/elasticsearch   ClusterIP   None             <none>        9200/TCP,9300/TCP   6h5m
service/head            ClusterIP   10.103.231.228   <none>        9100/TCP            3d8h
service/kibana          ClusterIP   10.102.83.26     <none>        5601/TCP            32h
service/logstash        NodePort    10.111.98.56     <none>        5040:30040/TCP      25m

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/head       1/1     1            1           3d8h
deployment.apps/kibana     1/1     1            1           32h
deployment.apps/logstash   3/3     3            3           25m

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/head-5c85b8d699       1         1         1       3d8h
replicaset.apps/kibana-5cfb7767fd     1         1         1       3h45m
replicaset.apps/logstash-746ddb77cc   3         3         3       25m

NAME                             READY   AGE
statefulset.apps/elasticsearch   3/3     6h5m
```

- 测试：

任选一个node ip，在本地添加hosts：

```a
192.168.30.130 elasticsearch.lzxlinux.cn
192.168.30.130 head.lzxlinux.cn
192.168.30.130 kibana.lzxlinux.cn
```

开启x-pack后访问head需要加上账号及密码，

```a
http://head.lzxlinux.cn/?auth_user=elastic&auth_password=elk-2021
```

直接使用 `http://elasticsearch:9200/` 连接es时会报错，使用 `http://elasticsearch.lzxlinux.cn/` 连接则正常。

![Image text](https://github.com/Tobewont/kubernetes/blob/master/elfk+xpack/img/elfk-1.png)

访问 `kibana.lzxlinux.cn`，账号/密码：`elastic`/`elk-2021`。

![Image text](https://github.com/Tobewont/kubernetes/blob/master/elfk+xpack/img/elfk-2.png)

访问nginx页面以产生日志：`curl ip:30080`，

![Image text](https://github.com/Tobewont/kubernetes/blob/master/elfk+xpack/img/elfk-3.png)

![Image text](https://github.com/Tobewont/kubernetes/blob/master/elfk+xpack/img/elfk-4.png)

访问tomcat页面以产生日志：`curl ip:30880`，

![Image text](https://github.com/Tobewont/kubernetes/blob/master/elfk+xpack/img/elfk-5.png)

![Image text](https://github.com/Tobewont/kubernetes/blob/master/elfk+xpack/img/elfk-6.png)

- kibana `@timestamp` 时区问题：

在kibana日志展示时，`@timestamp` 时区可能会与日志的时间戳不一致，`Stack Management` → `高级设置` → `Timezone for date formatting`，从 `Browser` 改为 `UTC` 即可。

如果仍无效，可以修改logstash filter 部分：

```bash
filter {
    grok {
        match => [ "message", "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level}" ]
    }

    date {
        match => [ "timestamp", "YYYY-MM-DD HH:mm:ss Z" ]
    }
}
```

之后在kibana上 `@timestamp` 即为当前时区。

另外，filebeat 如果不是通过 k8s 部署的，连接的 logstash 地址可以填公网 `ip:30040`，而 logstash 的配置文件中的 5040 端口不变。

至此，k8s部署 elfk 7.x + x-pack 完成。

---
