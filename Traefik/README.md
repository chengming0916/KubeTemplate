配置适用Traefik2.x

K3s启用Dashboard需要复制traefik-config.yaml到 /var/lib/rancher/k3s/server/manifests/
```
sudo cp traefik-config.yaml /var/lib/rancher/k3s/server/manifests/
```

然后重启k3s.service
```
sudo systemctl restart k3s.service
```

然后配置traefik service

```
kubectl apply -f traefik-service.yaml
```

最后配置IngressRoute
```
kubectl apply -f traefik-ingress.yaml
```