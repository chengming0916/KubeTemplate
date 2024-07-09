# 部署说明

#### NodePort方式暴露端口

```bash
# 部署pvc
kubectl apply -f gitea-pvc-local-path.yaml

# 部署config(暂时不用，空文件)
kubectl apply -f gitea-config.yaml

# 部署deploy
kubectl apply -f gitea-deployment.yaml

# 部署service,如果要使用22端口作为远程拉取代码端口需要本机修改ssh端口
kubectl apply -f gitea-service.yaml
```



#### Helm + Ingress 部署

```
helm upgrade gitea gitea/gitea --namespace gitea --install --create-namespace
```

