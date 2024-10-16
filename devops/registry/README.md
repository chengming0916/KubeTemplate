# 部署说明

```bash
# 创建namespace
kubectl create namespace devops

# 部署PVC
kubectl apply -f registry-pvc-local-path.yaml 

# 部署configmap
kubectl apply -f registry-config.yaml

# 部署deploy
kubectl apply -f registry-deployment.yaml

# 部署service,以NodePort方式暴露端口
kubectl apply -f registry-service.yaml
```

