### elfk

- 部署：

```bash
kubectl apply -f public-service-ns.yaml

kubectl apply -f elasticsearch/

kubectl apply -f kibana/

kubectl apply -f logstash/
```

filebeat以sidecar方式部署，每个应用的pod中包含filebeat容器。

filebeat收集不同应用的日志时，ConfigMap应该独立，避免出错。

- 示例：

此处以收集nginx和tomcat日志示例，

```bash
kubectl apply -f filebeat/
```

任选一个node ip，在本地添加hosts：

```a
192.168.30.130 kibana.lzxlinux.com
```

打开`kibana.lzxlinux.com`，访问kibana查看日志。

---
