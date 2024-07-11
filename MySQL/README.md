# 部署说明

```bash

# 部署PVC
kubectl apply -f mysql-pvc-local-path.yaml 

# 部署configmap
kubectl apply -f mysql-config.yaml

# 部署deploy
kubectl apply -f mysql-deployment.yaml

# 部署service,以NodePort方式暴露端口
kubectl apply -f mysql-service.yaml

```