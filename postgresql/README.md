# 部署说明

```bash

# 部署PVC
kubectl apply -f postgres-pvc-local-path.yaml 

# 部署configmap
kubectl apply -f postgres-config.yaml

# 部署deploy
kubectl apply -f postgres-deployment.yaml

# 部署service,以NodePort方式暴露端口
kubectl apply -f postgres-service.yaml

```