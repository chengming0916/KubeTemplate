# KubeTemplate

#### 介绍
K3s单机环境服务部署模板

#### 配置更新源
参考官方说明，需要在/etc/rancher/k3s/下新建registries.yaml
```bash
sudo cp registries.yaml /etc/rancher/k3s/registries.yaml
```

#### 配置自动补全
```bash
# 安装自动补全
sudo apt install bash-completion
source /usr/share/bash-completion/bash_completion

# 配置kubectl自动补全
kubectl completion bash | sudo tee /etc/bash_completion/kubectl > /dev/null

# 配置helm自动补全
helm completion bash | sudo tee /etc/bash_completion/helm > /dev/null
```

#### 配置自签名证书
```bash
helm install 
```

#### 配置Traefik
```bash
# 参考官方说明，配置K3s内置Traefik
# 复制到 /var/lib/rancher/k3s/server/manifests/ 自动生效
sudo cp traefik/traefik-config.yaml /var/lib/rancher/k3s/server/manifests/

# 重启k3s服务
sudo systemctl restart k3s.service


# 配置dashboard
kubectl apply -f traefik/traefik-ingress.yaml
```
访问https://traefik.example.io查看网页访问是否正常

#### 部署PostgreSQL
```bash
kubectl apply -f postgres/postgres-namespace.yaml
kubectl apply -f postgres/postgres-pvc-local-path.yaml
kubectl apply -f postgres/postgres-config.yaml
kubectl apply -f postgres/postgres-deployment.yaml
kubectl apply -f postgres/postgres-service.yaml
kubectl apply -f postgres/postgres-ingress.yaml
```

#### 部署Redis
```bash
kubectl apply -f redis/redis-namespace.yaml
kubectl apply -f redis/redis-config.yaml
kubectl apply -f redis/redis-pvc-local-path.yaml
kubectl apply -f redis/redis-deployment.yaml
kubectl apply -f redis/redis-service.yaml
kubectl apply -f redis/redis-ingress.yaml
```