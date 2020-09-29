### harbor

- nfs存储：

```bash
yum install -y nfs-utils rpcbind

mkdir -p /data/harbor/{chartmuseum,jobservice,registry,database,redis,trivy}

vim /etc/exports
```

```a
/data/harbor    192.168.30.0/24(rw,sync,no_root_squash)
```

```bash
chmod -R 777 /data/harbor

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
kubectl apply -f harbor-pv.yaml

kubectl get pv
 
NAME                 CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
harbor-chartmuseum   5Gi        RWO            Recycle          Available                                   7s
harbor-database      5Gi        RWO            Recycle          Available                                   7s
harbor-jobservice    1Gi        RWO            Recycle          Available                                   7s
harbor-redis         1Gi        RWO            Recycle          Available                                   7s
harbor-registry      10Gi       RWO            Recycle          Available                                   7s
harbor-trivy         5Gi        RWO            Recycle          Available                                   7s
```

- 部署：

```bash
kubectl apply -f public-service-ns.yaml

helm install myharbor harbor/ -n public-service

helm ls -n public-service

NAME    	NAMESPACE     	REVISION	UPDATED                                	STATUS  	CHART       	APP VERSION
myharbor	public-service	1       	2020-09-29 14:44:54.069561678 +0800 CST	deployed	harbor-1.5.0	2.1.0
```

```bash
kubectl get pvc -n public-service

NAME                                       STATUS   VOLUME               CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-myharbor-harbor-redis-0               Bound    harbor-redis         1Gi        RWO                           30s
data-myharbor-harbor-trivy-0               Bound    harbor-trivy         5Gi        RWO                           30s
database-data-myharbor-harbor-database-0   Bound    harbor-database      5Gi        RWO                           30s
myharbor-harbor-chartmuseum                Bound    harbor-chartmuseum   5Gi        RWO                           32s
myharbor-harbor-jobservice                 Bound    harbor-jobservice    1Gi        RWO                           32s
myharbor-harbor-registry                   Bound    harbor-registry      10Gi       RWO                           32s

kubectl get pod -n public-service
 
NAME                                             READY   STATUS    RESTARTS   AGE
myharbor-harbor-chartmuseum-c85bd9bc9-kvppc      1/1     Running   0          3m21s
myharbor-harbor-clair-7f5f4885d6-qk9t8           2/2     Running   3          3m21s
myharbor-harbor-core-56769ffc8d-52n6m            1/1     Running   0          3m22s
myharbor-harbor-database-0                       1/1     Running   0          3m22s
myharbor-harbor-jobservice-6c9cb4c87b-8wdvf      1/1     Running   0          3m22s
myharbor-harbor-notary-server-6f57f9d879-qd57m   1/1     Running   1          3m22s
myharbor-harbor-notary-signer-6df44c949b-l8vth   1/1     Running   1          3m22s
myharbor-harbor-portal-75dd5995b9-vgvzh          1/1     Running   0          3m21s
myharbor-harbor-redis-0                          1/1     Running   0          3m22s
myharbor-harbor-registry-5f5fd59b9b-npkbv        2/2     Running   0          3m22s
myharbor-harbor-trivy-0                          1/1     Running   0          3m22s
```

- 访问：

添加hosts：`harbor.lzxlinux.com`，使用初始账号密码 `admin`/`Harbor12345` 登录即可。

![Image text](https://github.com/Tobewont/kubernetes/blob/master/harbor/img/harbor-1.png)

![Image text](https://github.com/Tobewont/kubernetes/blob/master/harbor/img/harbor-2.png)

- docker添加harbor证书：

对于需要使用harbor仓库的节点，都要添加harbor证书。

```bash
kubectl get secrets -n public-service myharbor-harbor-ingress -o jsonpath="{.data.ca\.crt}" | base64 --decode
```

```a
-----BEGIN CERTIFICATE-----
MIIC9TCCAd2gAwIBAgIRAKImYyOICM1GmaHRbS98RMgwDQYJKoZIhvcNAQELBQAw
FDESMBAGA1UEAxMJaGFyYm9yLWNhMB4XDTIwMDkyOTA3MDA1MVoXDTIxMDkyOTA3
MDA1MVowFDESMBAGA1UEAxMJaGFyYm9yLWNhMIIBIjANBgkqhkiG9w0BAQEFAAOC
AQ8AMIIBCgKCAQEAwm0bFysRzMbGiAK31XWiZRhGx76qDLObGtF5xYpINfABkxC7
VglL66OC+iga2b1+MgLesdDkhW028jsrfPAnyl+S+xEgScytY9DKcRM8Ga72WYAw
DiKe9CdUhRtZ4o0SSCcmK38167R7R5YGpbuuoTmuBKXr5QE3FQot+yOZywpHHIBF
jhoz8LrvN2P3LKJlKhCi7mE+WPhbUrkDFo1iuYuaxN0bd9MOZvukNK3WqJh48zE4
33jcRLbwN79+731MeKE+hRswVEYbl2o6uqCP0cmJA6LFDyxVSCkcYyuo/ANoEw9G
bUga7IN2zI2vI7iCAxdLrfrECW7l0DeGih+g7wIDAQABo0IwQDAOBgNVHQ8BAf8E
BAMCAqQwHQYDVR0lBBYwFAYIKwYBBQUHAwEGCCsGAQUFBwMCMA8GA1UdEwEB/wQF
MAMBAf8wDQYJKoZIhvcNAQELBQADggEBAJANkTXThJebPlhxFLrG6xzN5J/Dgsiw
06GvLLlKU7ED34RlXaZ6YneQU47Zo67OtiO5NmAfu8Ts3Ak1fcEsmCAGWomL2+Ka
1zDilaFJA9le7L7wGHdusXt3isWpgebIC3x1lsjV5h3vIz2/9243gHR2Cic3mUnl
hFNzOChx2GKQASy0wq1Na2/ceoIRLL4vUJmtxmeRgyR7240+Sv9kcPEa2k9/Hp4k
IDAktVbB7J+mtiUPGQWD/53GZssDqadOlv9ShAZfU6uC9+tCSWgmrQJQqlxh2PZ2
Z/FPgVfkSR8nCJTohheWaHVLBECBwZHiGyNqiMJnQ4aqr1jMjErgnMw=
-----END CERTIFICATE-----
```

```bash
mkdir -p /etc/docker/certs.d/harbor.lzxlinux.com

cat <<EOF > /etc/docker/certs.d/harbor.lzxlinux.com/ca.crt
-----BEGIN CERTIFICATE-----
MIIC9TCCAd2gAwIBAgIRAKImYyOICM1GmaHRbS98RMgwDQYJKoZIhvcNAQELBQAw
FDESMBAGA1UEAxMJaGFyYm9yLWNhMB4XDTIwMDkyOTA3MDA1MVoXDTIxMDkyOTA3
MDA1MVowFDESMBAGA1UEAxMJaGFyYm9yLWNhMIIBIjANBgkqhkiG9w0BAQEFAAOC
AQ8AMIIBCgKCAQEAwm0bFysRzMbGiAK31XWiZRhGx76qDLObGtF5xYpINfABkxC7
VglL66OC+iga2b1+MgLesdDkhW028jsrfPAnyl+S+xEgScytY9DKcRM8Ga72WYAw
DiKe9CdUhRtZ4o0SSCcmK38167R7R5YGpbuuoTmuBKXr5QE3FQot+yOZywpHHIBF
jhoz8LrvN2P3LKJlKhCi7mE+WPhbUrkDFo1iuYuaxN0bd9MOZvukNK3WqJh48zE4
33jcRLbwN79+731MeKE+hRswVEYbl2o6uqCP0cmJA6LFDyxVSCkcYyuo/ANoEw9G
bUga7IN2zI2vI7iCAxdLrfrECW7l0DeGih+g7wIDAQABo0IwQDAOBgNVHQ8BAf8E
BAMCAqQwHQYDVR0lBBYwFAYIKwYBBQUHAwEGCCsGAQUFBwMCMA8GA1UdEwEB/wQF
MAMBAf8wDQYJKoZIhvcNAQELBQADggEBAJANkTXThJebPlhxFLrG6xzN5J/Dgsiw
06GvLLlKU7ED34RlXaZ6YneQU47Zo67OtiO5NmAfu8Ts3Ak1fcEsmCAGWomL2+Ka
1zDilaFJA9le7L7wGHdusXt3isWpgebIC3x1lsjV5h3vIz2/9243gHR2Cic3mUnl
hFNzOChx2GKQASy0wq1Na2/ceoIRLL4vUJmtxmeRgyR7240+Sv9kcPEa2k9/Hp4k
IDAktVbB7J+mtiUPGQWD/53GZssDqadOlv9ShAZfU6uC9+tCSWgmrQJQqlxh2PZ2
Z/FPgVfkSR8nCJTohheWaHVLBECBwZHiGyNqiMJnQ4aqr1jMjErgnMw=
-----END CERTIFICATE-----
EOF
```

```bash
echo '192.168.30.129 harbor.lzxlinux.com' >> /etc/hosts

docker login harbor.lzxlinux.com -u admin

Password:               # Harbor12345
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

可以看到登录成功，这种方式不需要重启docker。

- push插件：

```bash
helm plugin install https://github.com/chartmuseum/helm-push

helm plugin ls

NAME	VERSION	DESCRIPTION                      
push	0.8.1  	Push chart package to ChartMuseum
```

- 推送chart：

harbor新建项目`public`，

```bash
helm repo add myharbor --ca-file /etc/docker/certs.d/harbor.lzxlinux.com/ca.crt https://harbor.lzxlinux.com/chartrepo/public --username=admin --password=Harbor12345

helm repo ls

NAME    	URL                                                   
stable  	http://mirror.azure.cn/kubernetes/charts              
aliyun  	https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
harbor  	https://helm.goharbor.io                              
myharbor	https://harbor.lzxlinux.com/chartrepo/public
```

这里的 repo 的地址是 `<Harbor URL>/chartrepo/<项目名称>`，harbor 中每个项目是分开的 repo。

```bash
helm push harbor myharbor --ca-file /etc/docker/certs.d/harbor.lzxlinux.com/ca.crt
```

![Image text](https://github.com/Tobewont/kubernetes/blob/master/harbor/img/harbor-3.png)

helm部署harbor完成。

---
