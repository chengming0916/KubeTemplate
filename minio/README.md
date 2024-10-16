# 部署说明

```bash

# 部署PVC
kubectl apply -f minio-pvc-local-path.yaml 

# 部署configmap
kubectl apply -f minio-config.yaml

# 部署deploy
kubectl apply -f minio-deployment.yaml

# 部署service,以NodePort方式暴露端口
kubectl apply -f minio-service.yaml

```