# 部署说明

```bash

# 创建namespace
kubectl create namespace devops

# 部署PVC
kubectl apply -f harbor-pvc-local-path.yaml 

# 部署configmap
kubectl apply -f harbor-config.yaml

# 部署deploy
kubectl apply -f harbor-deployment.yaml

# 部署service,以NodePort方式暴露端口
kubectl apply -f harbor-service.yaml

```