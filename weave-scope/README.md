### weave-scope

- 部署：

```bash
kubectl apply -f weave-ns.yaml

kubectl apply -f manifests/
```

```bash
kubectl get svc -n weave

NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
weave-scope-app   ClusterIP   10.106.124.95   <none>        80/TCP    25s

kubectl get pod -n weave

NAME                                        READY   STATUS    RESTARTS   AGE
weave-scope-agent-27zpb                     1/1     Running   0          32s
weave-scope-agent-c5hcq                     1/1     Running   0          32s
weave-scope-agent-j4tf7                     1/1     Running   0          32s
weave-scope-agent-s8p6s                     1/1     Running   0          32s
weave-scope-app-bc7444d59-6xwkk             1/1     Running   0          33s
weave-scope-cluster-agent-5c5dcc8cb-4d7mh   1/1     Running   0          33s
```

- 访问ui：

添加hosts：`scope.lzxlinux.com`，访问`scope.lzxlinux.com`。

---
