### gitlab-runner

- nfs存储：

```bash
yum install -y nfs-utils rpcbind

mkdir -p /data/gitlab/{gitaly,minio,postgresql,redis}

vim /etc/exports
```

```a
/data/gitlab    192.168.30.0/24(rw,sync,no_root_squash)
```

```bash
chmod -R 755 /data/gitlab

exportfs -arv

systemctl enable rpcbind && systemctl start rpcbind

systemctl enable nfs && systemctl start nfs
```

nfs部署完毕。对于需要使用nfs的node节点，都要安装nfs：

```bash
yum install -y nfs-utils
```

- 创建pv：

```bash
kubectl apply -f gitlab-pv.yaml

kubectl get pv
 
NAME                CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
gitlab-gitaly       50Gi       RWO            Recycle          Available                                   2s
gitlab-minio        10Gi       RWO            Recycle          Available                                   2s
gitlab-postgresql   8Gi        RWO            Recycle          Available                                   2s
gitlab-redis        8Gi        RWO            Recycle          Available                                   2s
```

- 部署：

```bash
kubectl apply -f public-service-ns.yaml

kubectl apply -f shell/

kubectl apply -f shared-secrets/

kubectl apply -f registry/

kubectl apply -f redis/

kubectl apply -f postgresql/

kubectl apply -f minio/

kubectl apply -f sidekiq/

kubectl apply -f task-runner/

kubectl apply -f gitaly/

kubectl apply -f migrations/

kubectl apply -f webservice/

kubectl apply -f upgrade-check/
```

```bash
kubectl get pvc -n public-service

NAME                               STATUS   VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-gitlab-postgresql-0           Bound    gitlab-postgresql   8Gi        RWO                           10s
gitlab-minio                       Bound    gitlab-minio        10Gi       RWO                           12s
redis-data-gitlab-redis-master-0   Bound    gitlab-redis        8Gi        RWO                           10s
repo-data-gitlab-gitaly-0          Bound    gitlab-gitaly       50Gi       RWO                           10s

kubectl get svc -n public-service

NAME                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
gitlab-gitaly                ClusterIP   None             <none>        8075/TCP            41s
gitlab-minio-svc             ClusterIP   10.106.70.124    <none>        9000/TCP            41s
gitlab-postgresql            ClusterIP   10.99.82.121     <none>        5432/TCP            41s
gitlab-postgresql-headless   ClusterIP   None             <none>        5432/TCP            41s
gitlab-redis-headless        ClusterIP   None             <none>        6379/TCP            41s
gitlab-redis-master          ClusterIP   10.107.125.188   <none>        6379/TCP            41s
gitlab-registry              ClusterIP   10.111.0.240     <none>        5000/TCP            41s
gitlab-shell                 ClusterIP   10.108.217.155   <none>        22/TCP              41s
gitlab-webservice            ClusterIP   10.110.101.210   <none>        8080/TCP,8181/TCP   41s

kubectl get pod -n public-service

NAME                                         READY   STATUS      RESTARTS   AGE
gitlab-gitaly-0                              1/1     Running     0          3m31s
gitlab-migrations-1-2zf95                    1/1     Running     0          3m33s
gitlab-minio-7b8b675675-bs57b                1/1     Running     0          3m32s
gitlab-minio-create-buckets-1-zldwp          0/1     Completed   2          3m33s
gitlab-postgresql-0                          1/1     Running     1          3m31s
gitlab-redis-master-0                        1/1     Running     0          3m31s
gitlab-registry-6c7868fd7b-cncsd             1/1     Running     0          3m32s
gitlab-registry-6c7868fd7b-rvl4j             1/1     Running     0          3m32s
gitlab-shared-secrets-1-7d3-tmkjz            0/1     Completed   0          3m33s
gitlab-shell-68555f57cd-ns9tb                1/1     Running     0          3m32s
gitlab-sidekiq-all-in-1-v1-d87cff984-dpvwd   1/1     Running     0          3m32s
gitlab-task-runner-7d7b868c85-zcmt5          1/1     Running     0          3m32s
gitlab-upgrade-check-qnffj                   0/1     Completed   0          3m33s
gitlab-webservice-54b7d6cc6-h524f            2/2     Running     0          3m32s
```

- 访问：

```bash
kubectl get secrets -n public-service gitlab-initial-root-password -o yaml

apiVersion: v1
data:
  password: TEhpNVVubzNHZE5CbnZZWWtMUk9lTUlrMlZKS0VJY2dXdVoxazA5WlkxZU1iZEl1aEsxUDJzQ3g4YkRPV001dw==
kind: Secret
metadata:
  creationTimestamp: "2020-10-19T06:38:05Z"
  labels:
    app: gitlab
    component: shared-secrets
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:password: {}
      f:metadata:
        f:labels:
          .: {}
          f:app: {}
          f:component: {}
      f:type: {}
    manager: kubectl
    operation: Update
    time: "2020-10-19T06:38:06Z"
  name: gitlab-initial-root-password
  namespace: public-service
  resourceVersion: "239043"
  selfLink: /api/v1/namespaces/public-service/secrets/gitlab-initial-root-password
  uid: 8ca53b05-ff23-4f1f-9d1b-e5ebe82f45ff
type: Opaque
```

```bash
echo 'TEhpNVVubzNHZE5CbnZZWWtMUk9lTUlrMlZKS0VJY2dXdVoxazA5WlkxZU1iZEl1aEsxUDJzQ3g4YkRPV001dw==' | base64 -d

LHi5Uno3GdNBnvYYkLROeMIk2VJKEIcgWuZ1k09ZY1eMbdIuhK1P2sCx8bDOWM5w
```

添加hosts：`gitlab.lzxlinux.com`，默认账号：`root`，密码：`LHi5Uno3GdNBnvYYkLROeMIk2VJKEIcgWuZ1k09ZY1eMbdIuhK1P2sCx8bDOWM5w`。

![Image text](https://github.com/Tobewont/kubernetes/blob/master/gitlab/img/gitlab-1.png)

![Image text](https://github.com/Tobewont/kubernetes/blob/master/gitlab/img/gitlab-2.png)

k8s部署gitlab完成。

---
