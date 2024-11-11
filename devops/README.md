# 基于K3s的DevOps环境搭建

### 安装K3s
参考[K3s安装](https://chengming0916.github.io)

### 配置CoreDNS
```bash
kubectl apply -f Devops/coredns-custom.yaml
```



### 部署cert-manager

```bash
helm repo add jetstack https://charts.jetstack.io

helm upgrade cert-manager jetstack/cert-manager \
    --namespace cert-manager \
    --install --create-namespace \
    --set crds.enabled=true

# 自签名证书
kubectl apply -f selfsigned-cluster-issuer.yaml

# letsencrypt证书
kubectl apply -f letsencrypt-issuer-staging.yaml

```

内网使用自签名证书，然后参考[K3s导出证书](https://chengming0916.github.io/2024/04/02/Kubernetes/K3s导出证书/)，导出根证书，在需要使用的机器上导入证书。

https://www.teanote.pub/archives/532

测试证书是否可用

```bash
kubectl apply -f DevOps/test/whoami-deployment.yaml
kubectl apply -f DevOps/test/whoami-service.yaml
kubectl apply -f DevOps/test/whoami-cert.yaml
kubectl apply -f DevOps/test/whoami-ingress.yaml
```

访问https://whoami.example.io查看是否有证书

### 配置Traefik

```bash
sudo cp DevOps/traefik/traefik-config.yaml /var/lib/rancher/k3s/mainfest/
sudo systemctl restart k3s.service
```



### 测试

```bash
kubectl apply -f DevOps/test/whoami-cert.yaml
kubectl apply -f DevOps/test/whoami-deployment.yaml
kubectl apply -f DevOps/test/whoami-service.yaml
kubectl apply -f DevOps/test/whoami-ingress.yaml
```



### 部署Rancher

```bash
# 添加Helm Chart库
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable

# 安装
helm upgrade rancher rancher-stable/rancher \
    --namespace cattle-system \
    --create-namespace --install \
    -f DevOps/rancher/rancher-values.yaml
```



### 部署PostgreSQL

```bash

```



### 部署Redis

```

```



### 部署Harbor

```bash

```



### 部署Gitea

```bash

```



### 部署Drone

```bash

```



### 部署SonarQube

```bash

```

