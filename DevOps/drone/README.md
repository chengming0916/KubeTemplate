# 部署说明

**配置Gitea登录失败，版本Gitea:1.22,Drone:2**

### 部署前需要创建postgres用户和数据库

```sql
CREATE USER drone WITH PASSWORD 'drone';
CREATE DATABASE drone WITH OWNER drone;
```

### 配置Gitea OAuth2登录

参考[Gitea | Drone](https://docs.drone.io/server/provider/gitea/)

### 部署Drone

```bash
# 创建namespace
kubectl create namespace devops

# 部署PVC
kubectl apply -f drone-pvc-local-path.yaml 

# 部署configmap
kubectl apply -f drone-config.yaml

# 部署deploy
kubectl apply -f drone-deployment.yaml

# 部署service,以NodePort方式暴露端口
kubectl apply -f drone-service.yaml

# 使用Traefik暴露服务
kubectl	apply -f drone-ingress.yaml
```