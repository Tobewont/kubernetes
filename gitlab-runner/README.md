### gitlab-runner

- 查看注册url和token：

![Image text](https://github.com/Tobewont/kubernetes/blob/master/gitlab-runner/img/runner-1.png)

- base64：

```bash
echo -n 'HtyY1AEoI5KsRjJzOZAA14ekUJ8aJiA1Cc049cAZC0MDFY5EyN0kOcvbTwnCG7sl' | base64             #base64加密

SHR5WTFBRW9JNUtzUmpKek9aQUExNGVrVUo4YUppQTFDYzA0OWNBWkMwTURGWTVFeU4wa09jdmJUd25DRzdzbA==
```

```bash
echo 'SHR5WTFBRW9JNUtzUmpKek9aQUExNGVrVUo4YUppQTFDYzA0OWNBWkMwTURGWTVFeU4wa09jdmJUd25DRzdzbA==' | base64 -d             #base64解密

HtyY1AEoI5KsRjJzOZAA14ekUJ8aJiA1Cc049cAZC0MDFY5EyN0kOcvbTwnCG7sl
```

加解密工具：http://tool.chinaz.com/tools/base64.aspx

- 部署：

```bash
kubectl apply -f public-service-ns.yaml

kubectl apply -f gitlab-runner-rbac.yaml

kubectl apply -f gitlab-runner-secret.yaml

kubectl apply -f gitlab-runner-cm.yaml

kubectl apply -f gitlab-runner-deploy.yaml
```

```bash
kubectl get pod -n public-service | grep gitlab-runner

gitlab-runner-5dddb4498f-dvr4r                  1/1     Running     0          22s
```

![Image text](https://github.com/Tobewont/kubernetes/blob/master/gitlab-runner/img/runner-2.png)

k8s部署gitlab-runner完成。

---
