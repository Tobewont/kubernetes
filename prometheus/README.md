### prometheus

- 部署：

```bash
kubectl apply -f public-service-ns.yaml

kubectl apply -f node-exporter/

kubectl apply -f k8s-components/

kubectl apply -f kube-state-metrics/

kubectl apply -f blackbox-exporter/

kubectl apply -f dingtalk/

kubectl apply -f alertmanager/

kubectl apply -f prometheus/

kubectl apply -f grafana/
```

```bash
kubectl get all -n public-service 

NAME                                      READY   STATUS    RESTARTS   AGE
pod/alertmanager-9c4bf8565-z9mp9          1/1     Running   0          2m54s
pod/blackbox-exporter-57d847fc4c-mq8mx    1/1     Running   0          2m58s
pod/dingtalk-957f5896-9bd9b               1/1     Running   0          2m56s
pod/grafana-76779dc8cf-2fk4x              1/1     Running   0          2m46s
pod/kube-state-metrics-5d5f7cd774-tw4sw   1/1     Running   0          2m58s
pod/node-exporter-29bkg                   1/1     Running   0          3m5s
pod/node-exporter-45k2d                   1/1     Running   0          3m5s
pod/node-exporter-8dbts                   1/1     Running   0          3m5s
pod/node-exporter-9kwwt                   1/1     Running   0          3m5s
pod/node-exporter-bxhcf                   1/1     Running   0          3m5s
pod/prometheus-65848cf9b4-m5kcf           1/1     Running   0          2m49s

NAME                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/alertmanager         ClusterIP   10.98.52.72      <none>        9093/TCP         2m55s
service/blackbox-exporter    NodePort    10.106.73.127    <none>        9115:30115/TCP   2m58s
service/dingtalk             ClusterIP   10.103.205.136   <none>        8060/TCP         2m57s
service/grafana              ClusterIP   10.103.12.113    <none>        3000/TCP         2m47s
service/kube-state-metrics   ClusterIP   10.98.99.215     <none>        8080/TCP         3m1s
service/node-exporter        ClusterIP   None             <none>        9100/TCP         3m6s
service/prometheus           ClusterIP   10.99.50.109     <none>        9090/TCP         2m51s

NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/node-exporter   5         5         5       5            5           <none>          3m5s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/alertmanager         1/1     1            1           2m55s
deployment.apps/blackbox-exporter    1/1     1            1           2m58s
deployment.apps/dingtalk             1/1     1            1           2m57s
deployment.apps/grafana              1/1     1            1           2m46s
deployment.apps/kube-state-metrics   1/1     1            1           3m
deployment.apps/prometheus           1/1     1            1           2m51s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/alertmanager-9c4bf8565          1         1         1       2m55s
replicaset.apps/blackbox-exporter-57d847fc4c    1         1         1       2m58s
replicaset.apps/dingtalk-957f5896               1         1         1       2m56s
replicaset.apps/grafana-76779dc8cf              1         1         1       2m46s
replicaset.apps/kube-state-metrics-5d5f7cd774   1         1         1       3m
replicaset.apps/prometheus-65848cf9b4           1         1         1       2m51s
```

任选一个node ip，在本地添加hosts：

```a
192.168.30.130 alertmanager.lzxlinux.com
192.168.30.130 prometheus.lzxlinux.com
192.168.30.130 grafana.lzxlinux.com
```

配置grafana：数据源是`http://prometheus:9090`，导入`主机详情`模板8919。

---

- grafana配置kubernetes数据源：

启动插件：`Plugins` → `kubernetes` → `Enable`，然后配置集群访问地址及访问证书：

如果是通过kubeadm方式搭建的k8s集群，会有一个 `/etc/kubernetes/admin.conf` 文件，里面包含了客户端的证书和密码base64编码。

```bash
cat /etc/kubernetes/admin.conf
```

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01EVXhNakV3TURBek1Gb1hEVE13TURVeE1ERXdNREF6TUZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTDVpCmYxTGcxUUlqN0VlWlZ0cVFmS3dGZjg4V3NVbVVialZldll5NDZTUittMWpwRWdvM2wxWEIvZHBFNzRWOGtqTGQKZkdGdmVWZkVxNy8rMzdyamNGMXRpSm1BbThLZnMrMW9QdEpLOE0yZjNTSm5FZVVIQUlBeFl2cUE4ZFNsbThTQwpmSkJWU2J3K1pROTBTelpKNzdQUzFuZTBmYnRod0Y2VHE0Uy9FV3h3cUZZMzF5cENub05lVUNtcElsSjVnYWdtCnJ2QmhkTmFNb2oyQlRrMWNDVjh3dkRVS3RlbXFVYVE4R2ZCalZLeHhkdWtwcjJ3S3RPbXZkem1vMEdLSE11MFcKWmQ1TVd0dStIQVZrTXhzcE95Yk41NkFkNnloUkN5YkFJbTN2ZWJlTFV5cjBEY2JhNzJXNVlPRHRCY3ZBOEJxOAoxR1JQc1EwaXBUdGtYbDVCZEhzQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFJS0lIb25wVllFWWpwR3JrN2wraGJyeGlxZXkKeGFQT1M3UW5TZEVZMC94TWtiUWxKcy9rUFcxU2lVemdoUk4wQWJxMnFtTXVuNHhlZ0pLdGVPNXhYRGJZNEhZbgpVVCtPWG0rQ1hBQjd3S3pYcDlmUTZBUDk3cmY0L2FRaXlGZEtsZUJ6Y3JNUkErZHZWTjk3NGlHUW94aFh3T1FNCmZXeGNrMDNhU0Qvc2s5UnJrcFhlL1g2NHQrV3BkUlFGRjE2YVFlSHVxNnJQRWZTR2VPUWVpcVIrQVgvdWpIOHoKZzJZY2JKWE85U3ZheXcyb3oxSlozTUx6K0FpeE5RTHFNYU00Tm43TklvMExxUHFqNzZoU3d1Qk1nREE0VnFtZAowZHRtS211OVZjTGZHcW9ITnZnajlTYlVlZ1crL3VEbzcwVXdvb2NGTmlnSnRnOVVSZWpEUXJJSm4rUT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://192.168.30.188:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM4akNDQWRxZ0F3SUJBZ0lJZWtMYS9Fc1ZDSGd3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBMU1USXhNREF3TXpCYUZ3MHlNVEExTVRJeE1EQXdNekphTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXJ2a084MUg0ZW9zak5kM3oKSy9UUEhHcGtCR1FvZm1hbm9ldjRlWXNmUTlPZW0wYzBvVUJ3cXoxM2JabmJUbmJweFFqbmdZMkc4bHF4UmkwaQpCdlA2ZmtmS0ZFQlZzUTd4dGlqZXBrdnByWEdPL08wUUE1U0k4NHJzTjVHOVhOa2pQbWdzYTBlblZxNUVvRTBGClRaNXpRRjlwUlkxWUZZZXYrTDE1bU5FaXlScUg4UDJRY3BoUmxWK09IUXVHaVdLNEhIRVB2QWw2QUpJeWN6d3MKWWMrdk1IdHlZbmF5NUMwUldVWHhyUmc0ZytKMksrY1h1YlF0elhXdjdxaTNhNjFDekpaZi9TZkNOd0Jyam9zRwp0b215WEJWNVZTVGJUYVk1OFZrLzFPK1NSc3BybjF3TDc0djdXUXVEaE9ydXhBRXpuYmRXWWxOMEZBMm5MTjlZCmwxWkVKUUlEQVFBQm95Y3dKVEFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFHazlIRDAxRmRRUnd4THhGUi8yRjdPM2ZpdGRFV3pDTC9UawpsZUxZaGlQaVh3NjNwOGtWU0VabEIyNEYzNEd2WlB3YS9LWnNUQnZXM0Mwek9uNGpHQ2hueHEvaVdqTWFnVEdBCktPUFV2bUI2VzhvVzhlb0lrSStOOEs0NFhSRnZzeGIwNUtqaCtwd0VZZzJUQXpBNEFlQzlnSjZYaTBzbHpnVnIKcWRzbXZtV0QzNEdXYzJOcVIzSDA3cW43RlJwRHIrTjlrTHE4Ukt4L0YwMWNCV1I3VVRZcnJTLzJEQ2t1N3lsWgptdTcwcXZicndYWnF6TkI5b05hQk82SHJsZXpuU2JQbnFKZUo0Q1czc2NMNmJ1N3A3bEppV1VQb0VHT0xic3YvCnFjT0xqdnZSRFF6eC9Xak5DWFZLNFhxbzJjVERGYitXeFJ1U2xGaUlQclk1QjlkQlFJWT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBcnZrTzgxSDRlb3NqTmQzeksvVFBIR3BrQkdRb2ZtYW5vZXY0ZVlzZlE5T2VtMGMwCm9VQndxejEzYlpuYlRuYnB4UWpuZ1kyRzhscXhSaTBpQnZQNmZrZktGRUJWc1E3eHRpamVwa3ZwclhHTy9PMFEKQTVTSTg0cnNONUc5WE5ralBtZ3NhMGVuVnE1RW9FMEZUWjV6UUY5cFJZMVlGWWV2K0wxNW1ORWl5UnFIOFAyUQpjcGhSbFYrT0hRdUdpV0s0SEhFUHZBbDZBSkl5Y3p3c1ljK3ZNSHR5WW5heTVDMFJXVVh4clJnNGcrSjJLK2NYCnViUXR6WFd2N3FpM2E2MUN6SlpmL1NmQ053QnJqb3NHdG9teVhCVjVWU1RiVGFZNThWay8xTytTUnNwcm4xd0wKNzR2N1dRdURoT3J1eEFFem5iZFdZbE4wRkEybkxOOVlsMVpFSlFJREFRQUJBb0lCQUI4OHI0S1krN2RFNThCUwpJM3VSZFBncHRqbGllQ2c0dzJ5UTZBY3E0eVlFdmFnVENqNVBkczNiWjFyVndPVTlMWGJUcENEbzExS2xCa2owCi9jSW9CR3hPL0xDbzI2T0VlM3A5eVdIKzQzVG5kUk9LYnZWMHF3NXZtc1JBN0lHSzhsUE4zVUE1eHBJZkFubHIKeHFxWXd4S1c5Z0JJdjVUNGFGNEwxWTJHcUtNbUlhenVjLzVleU5rZjk3bnRyOFJncXQxcDJyaWJIVS9nRzFlYgpIWktyNm01UWx2MWJpYTFIYms2SWI2b1pYTVFIWWJSckpVemJSaWp6eE1RVkRmZ2tpalhSR3pSRjdZeVFZbTk5CmwwUzI1bDYzY1dIT3J6czM5R21xRmQ4Z0JJc0M3SkxuUThUY01nb1Axb0M5WXUzMGRrSDBvUi95M2lOTnFxRW8KZVJ2d0w0RUNnWUVBeG1tY2JDZXRlRDI5VWZDTE1kKzByTU14aVV4bHI3TTdrYUJOeWxPK2lOZWxKbUk5UHRXcwpkUzQzS2hkeElnSVNIQVRTdWxpR3VKMStEekxTWWNGZ2FkSmQxd25NNHppOUc2cW9NOXZTTVN2ZGtvK28ydXRqCmNscVBZcnVRbC9nK252dkE2N1ZyZzAyUXF0NFlqcmkrYmxQL1RNaDZGS3NGa0VQeXZPTHUxOTBDZ1lFQTRjSFgKUm43WUl2TWtMNGQrR3dkUmRwcXl5YStxVC9nbUtKTnNwb3VLUVZlaUd3aW9vR21BR0E0MEJBR2hyN24vMXB6Rwo5VkVQb201VDdPRnVmVWxGaUNURmJBblN2RWU5RTREUHJ3SDNhazlXR0JzcWxYcUZwMjdwWWFyZ3NSS2JDWU9UCm9Nc1FJR0wxelN4NEpkdFArMUxDQ1BuRnowMTNkajhRbmc4TVBPa0NnWUFwczF5cTVwUHc1NWo0dGNPcmtjYloKWUpUeXRGblMyYXExYXFtdTBuY0RMNytJRjdHam1Ta0wzOUM4U2Z6L0ZzeFRremZ1N2xneVNQZUxualRWVXQwKwpvSFlVa2Z5NzdOcmlDN1lhWUNNSExwNzlCTENLZ2xwK1dFWTJqQkZSdjF6NThST1U5cVpJREc5UldpaHpKcVR2CmJ6d0RHVWQvUElxSXpaOGd6OWsvQ1FLQmdIbEFRaDVEdkZReElNdENTM0c2NFg4QklXdC9wTXFrcmVIM0pGRGkKKzFPUy9LYm1aS01iWnNnRXdOMHgveVJCa3U0eWNBMk1Cd2lubHYzUUtpYXlOdDBqV3NGbkdUODBqSkd3Q2x1bApnN3dlZGxBbUx4M3ZtMTlOQzU0QVNBUHl5VUEzNGc5bllQYjBENjZ0NXEzMmQ2TzFWQys3N3dralF6bElMK1drCmtWOFpBb0dBVk01R1lLbnpNVjUzVzNXT3I0dFdLSm5XUHFiaHVlUEt5SXMzbTNkUzhGUE56SDU2UHhNKzRUM24Ka2NzT1VsZTlkQkFENXRXT3E5eHFmNWF4MXpaU2s1SzFhdUphSzRaa3RzNkdMRUgrU09WckdoK1JXQWtRcUFVbgo0Qmk4ZVA4MmR5M3N2RmV1UkNvTWFXRVQ0QlFHaGRQaFFCd1NNdlYrSWI2R3U0VldwN289Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
```

其中属性`certificate-authority-data`、`client-certificate-data`、`client-key-data` 对应 CA 证书、Client 证书、Client 私钥，文件里面的内容是base64编码过后的，分别执行 `echo "<base64 code>" | base64 -d` 就能还原成证书源文件。

首先将`Datasource`设置为`Prometheus`，点击`Save`保存即可，grafana中自动出现k8s相关的dashboard，

dashboard中有部分显示异常，这是因为prometheus的metric有更新，比如`node_cpu`改成了`node_cpu_seconds_total`。

可以重新定义该dashboard的变量（可以参考8919 dashboard的变量），然后根据prometheus界面的metrics来调试。

---
