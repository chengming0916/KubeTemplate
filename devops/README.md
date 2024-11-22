# 基于K3s的DevOps环境搭建

### 安装K3s
参考[K3s安装](https://chengming0916.github.io)

### 配置CoreDNS
```bash
kubectl apply -f coredns-custom.yaml
```

### 部署cert-manager
参考[cert-manager 配置 自签名ca以签发证书](https://www.teanote.pub/archives/532)
```bash
helm repo add jetstack https://charts.jetstack.io

helm upgrade cert-manager jetstack/cert-manager \
    --namespace cert-manager \
    --install --create-namespace \
    --set crds.enabled=true

# 自签名证书
kubectl apply -f selfsigned-cluster-issuer.yaml

kubectl -n cert-manager get secrets       # 查看证书
kubectl -n cert-manager get ClusterIssuer # 查看证书机构

# letsencrypt证书
kubectl apply -f letsencrypt-issuer-staging.yaml
```

内网使用自签名证书，然后参考[K3s导出证书](https://chengming0916.github.io/2024/04/02/Kubernetes/K3s导出证书/)，导出根证书，在需要使用的终端导入证书。


### 配置Traefik

```bash
sudo cp devops/traefik/traefik-config.yaml /var/lib/rancher/k3s/mainfest/
sudo systemctl restart k3s.service
```

### 测试证书是否可用

```bash
kubectl apply -f ../whoami/whoami-deployment.yaml
kubectl apply -f ../whoami/whoami-service.yaml
kubectl apply -f ../whoami/whoami-cert.yaml
kubectl apply -f ../whoami/whoami-ingress.yaml
```

访问 https://whoami.example.io 查看是否有证书

### 部署PostgreSQL

```bash
kubectl create ns postgres 
kubectl apply -f ../postgres/postgres-pvc-local-path.yaml
kubectl apply -f ../postgres/portgres-config.yaml
kubectl apply -f ../postgres/postgres-deployment.yaml
kubectl apply -f ../postgres/posrgres-service.yaml
kubectl apply -f ../postgres/postgres-ingress.yaml
```

### 部署Redis

```bash
kubectl create ns redis
kubectl apply -f ../redis/redis-pvc-local-path.yaml
kubectl apply -f ../redis/redis-config.yaml
kubectl apply -f ../redis/redis-deployment.yaml
kubectl apply -f ../redis/redis-service.yaml
kubectl apply -f ../redis/redis-ingress.yaml
```

### 部署Harbor

```bash
helm repo add harbor https://helm.goharbor.io
helm upgrade harbor harbor/harbor --namespace harbor \
    --create-namespace --install \
    -f harbor-values-with-ingress.yaml
```
默认用户名`admin`, 密码`Harbor123456`,可在配置中自定义密码。

docker配置私有镜像源
```bash
sudo mkdir -p /etc/docker/certs.d/harbor.example.io
sudo cp root-ca.crt /etc/docker/certs.d/harbor.example.io
sudo update-ca-trust extract  # 执行更新使证书生效
```

docker配置文件示例
```json
{
    "registry-mirrors": [
        "https://docker.m.daocloud.io",
        "https://.bei-jing.aliyuncs.com",
        "https://tencentyun.com"
    ],
    "insecure-registries": [
        "https://harbor.example.io"
    ]
}
```

只有登录成功才能推送镜像，所以配置完证书后尝试登录：
```bash
docker login -u admin -p Harbor123456 harbor.example.io
```

### 部署Gitea

依赖PostgreSQL

```SQL
// 创建用户
CREATE USER gitea WITH PASSWORD 'gitea';

// 创建数据库
CREATE DATABASE gitea WITH OWNER gitea; 
```

```bash
kubectl create ns gitea
kubectl apply -f gitea/gitea-pvc-local-path.yaml
kubectl apply -f gitea/gitea-config.yaml
kubectl apply -f gitea/gitea-deployment.yaml
kubectl apply -f gitea/gitea-service.yaml
kubectl apply -f gitea/gitea-ingress.yaml
```

helm部署
```bash
helm repo add gitea https://
helm upgrade gitea gitea/gitea --namespace gitea \
    --install --create-namespace -f gitea-values.yaml
```

### 部署Drone

```SQL
CREATE USER drone WITH PASSWORD 'drone';
CREATE DATABASE drone WITH OWNER drone;
```

```bash
kubectl create ns drone 
kubectl apply -f drone/drone-pvc-local-path.yaml
kubectl apply -f drone/drone-config.yaml
kubectl apply -f drone/drone-deployment.yaml
kubectl apply -f drone/drone-service.yaml
kubectl apply -f drone/drone-ingress.yaml
```
配置Gitea OAuth2登录

**必须Gitea和Drone链接为HTTPS,否则可能配置失败**

参考[Gitea | Drone](https://docs.drone.io/server/provider/gitea/)

### 部署SonarQube

```SQL
CREATE USER sonar WITH PASSWORD 'sonar';
CREATE DATABASE sonar WITH OWNER sonar;
```

```bash
kubectl create ns sonar
kubectl apply -f drone/drone-pvc-local-path.yaml
kubectl apply -f drone/drone-config.yaml
kubectl apply -f drone/drone-deployment.yaml
kubectl apply -f drone/drone-service.yaml
kubectl apply -f drone/drone-ingress.yaml
```

### 部署Docker Registry
可选，功能与Harbor重复
```bash
kubectl apply -f registry/registry-pvc-local-path.yaml
kubectl apply -f registry/registry-config.yaml
kubectl apply -f registry/registry-deployment.yaml
kubectl apply -f registry/registry-service.yaml
```