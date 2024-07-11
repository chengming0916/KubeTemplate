# 部署说明

```bash

# 部署PVC
kubectl apply -f redis-pvc-local-path.yaml 

# 部署configmap
kubectl apply -f redis-config.yaml

# 部署deploy
kubectl apply -f redis-deployment.yaml

# 部署service,以NodePort方式暴露端口
kubectl apply -f redis-service.yaml

```