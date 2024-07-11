### 准备环境

#### 创建命名空间
```bash
kubectl create namespace harbor
```
#### 创建存储

##### 使用NFS

```bash
# 挂载NFS
mount -o 192.168.0.2:/nfs /nfs
# 创建文件夹
mkdir -p /nfs/harbor/registry
mkdir -p /nfs/harbor/chartmuseum
mkdir -p /nfs/harbor/jobservice
mkdir -p /nfs/harbor/database
mkdir -p /nfs/harbor/redis
mkdir -p /nfs/harbor/trivy

# 部署PV
kubectl apply -f harbor-pv-nfs.yaml

# 部署PVC pvc默认启用local-path
kubectl apply -f harbor-pvc.yaml
```

harbor-pv.yaml

```yaml
#registry-PV
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-registry
  namespace: harbor
  labels:
    app: harbor-registry
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: "harbor"
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /nfs/harbor/registry
    server: 192.168.0.2
---
#harbor-chartmuseum-pv (不使用helm chart可省略)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-chartmuseum
  namespace: harbor
  labels:
    app: harbor-chartmuseum
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: "harbor"
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /nfs/harbor/chartmuseum
    server: 192.168.0.2
---
#harbor-jobservice-pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-jobservice
  namespace: harbor
  labels:
    app: harbor-jobservice
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: "harbor"
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /nfs/harbor/jobservice
    server: 192.168.0.2
---
#harbor-database-pv (使用外部数据库可省略)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-database
  namespace: harbor
  labels:
    app: harbor-database
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: "harbor"
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /nfs/harbor/database
    server: 192.168.0.2
---
#harbor-redis-pv (使用外部Redis可省略)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-redis
  namespace: harbor
  labels:
    app: harbor-redis
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: "harbor"
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /nfs/harbor/redis
    server: 192.168.0.2
---
#harbor-trivy-pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: harbor-trivy
  labels:
    app: harbor-trivy
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: "harbor"
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /nfs/harbor/trivy
    server: 192.168.0.2
```

harbor-pvc-nfs.yaml

```yaml
#harbor-registry-pvc
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: harbor-registry
  namespace: harbor
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: "harbor"
  resources:
    requests:
      storage: 100Gi
  selector:
    matchLabels:
      app: harbor-registry
---
#harbor-chartmuseum-pvc
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: harbor-chartmuseum
  namespace: harbor
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: "hub"
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      app: harbor-chartmuseum
---
#harbor-jobservice-pvc
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: harbor-jobservice
  namespace: harbor
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: "hub"
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      app: harbor-jobservice 
---
#harbor-database-pvc
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: harbor-database
  namespace: harbor
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: "hub"
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      app: harbor-database  
---
#harbor-redis-pvc
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: harbor-redis
  namespace: harbor
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: "hub"
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      app: harbor-redis
---
#harbor-trivy-pvc
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: harbor-trivy
  namespace: harbor
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: "hub"
  resources:
    requests:
      storage: 5Gi
  selector:
    matchLabels:
      app: harbor-trivy
```

##### 使用LocalPath(K3s)

```

```


#### 创建证书

##### 使用cert-manager生成证书

参考

##### 使用自定义证书

###### 生成证书文件

下面执行步骤时，需要输入一些证书信息，其中 Common Name 必须要设置为和你要给 Harbor 的域名保持一致，如 Common Name (eg, your name or your server's hostname) []:example.io。

```
## 获得证书
$ openssl req -newkey rsa:4096 -nodes -sha256 -keyout ca.key -x509 -days 3650 -out ca.crt

## 生成证书签名请求
$ openssl req -newkey rsa:4096 -nodes -sha256 -keyout tls.key -out tls.csr

## 生成证书
$ openssl x509 -req -days 3650 -in tls.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out tls.crt
```

###### 生成 Secret 资源

创建 Kubernetes 的 Secret 资源，且将证书文件导入：

- -n：指定创建资源的 Namespace
- --from-file：指定要导入的文件地址

```bash
kubectl create secret generic harbor-tls --from-file=tls.crt --from-file=tls.key --from-file=ca.crt -n harbor

查看是否创建成功
kubectl get secret harbor-tls -n harbor
kubectl describe secret harbor-tls -n harbor
```



### 安装

```bash
# 添加Helm仓库
helm repo add harbor https://helm.goharbor.io
# 部署Harbor
helm upgrade harbor harbor/harbor \
	--namespace harbor --install \
	--create-namespace -f harbor-values.yaml 
	
# 查看部署是否完成
kubectl get deployment -n harbor
```



harbor-values.yaml

```yaml
#Ingress 网关入口配置
expose:
  type: ingress
  tls:
    ### 是否启用 https 协议，如果不想启用 HTTPS，则可以设置为 false
    enabled: true       
    ### 指定使用 sectet 挂载证书模式，且使用上面创建的 secret 资源
    certSource: secret
    secret:
      secretName: "harbor-tls"
      notarySecretName: "harbor-tls"
  ingress:                          
    hosts:
      ### 配置 Harbor 的访问域名，需要注意的是配置 notary 域名要和 core 处第一个单词外，其余保持一致
      core: hub.mydlq.club   
      notary: notary.mydlq.club
    controller: default
    annotations:
      ingress.kubernetes.io/ssl-redirect: "true"
      ingress.kubernetes.io/proxy-body-size: "0"
      #### 如果是 traefik ingress，则按下面配置：
      kubernetes.io/ingress.class: "traefik"
      traefik.ingress.kubernetes.io/router.tls: 'true'
      traefik.ingress.kubernetes.io/router.entrypoints: websecure
      #### 如果是 nginx ingress，则按下面配置：
      #nginx.ingress.kubernetes.io/ssl-redirect: "true"
      #nginx.ingress.kubernetes.io/proxy-body-size: "0"
  ## 如果不想使用 Ingress 方式，则可以配置下面参数，配置为 NodePort                  
  #clusterIP:
  #  name: harbor
  #  ports:
  #    httpPort: 80
  #    httpsPort: 443
  #    notaryPort: 4443
  #nodePort:
  #  name: harbor
  #  ports:
  #    http:
  #      port: 80
  #      nodePort: 30011
  #    https: 
  #      port: 443
  #      nodePort: 30012
  #    notary: 
  #      port: 4443
  #      nodePort: 30013

## 如果Harbor部署在代理后，将其设置为代理的URL，这个值一般要和上面的 Ingress 配置的地址保存一致
externalURL: https://hub.mydlq.club

### Harbor 各个组件的持久化配置，并设置各个组件 existingClaim 参数为上面创建的对应 PVC 名称
persistence:
  enabled: true
  ### 存储保留策略，当PVC、PV删除后，是否保留存储数据
  resourcePolicy: "keep"    
  persistentVolumeClaim:
    registry:
      existingClaim: "harbor-registry"
      size: 100Gi
    chartmuseum:
      existingClaim: "harbor-chartmuseum"
      size: 5Gi
    jobservice:
      existingClaim: "harbor-jobservice"
      size: 5Gi
    database:
      existingClaim: "harbor-database"
      size: 5Gi
    redis:
      existingClaim: "harbor-redis"
      size: 5Gi
    trivy:
      existingClaim: "harbor-trivy"
      size: 5Gi

### 默认用户名 admin 的密码配置，注意：密码中一定要包含大小写字母与数字
harborAdminPassword: "Mydlq123456"

### 设置日志级别
logLevel: info

#各个组件 CPU & Memory 资源相关配置
nginx:
  resources:
    requests:
      memory: 256Mi
      cpu: 500m
portal:
  resources:
    requests:
      memory: 256Mi
      cpu: 500m  
core:
  resources:
    requests:
      memory: 256Mi
      cpu: 1000m
jobservice:
  resources:
    requests:
      memory: 256Mi
      cpu: 500m
registry:
  registry:
    resources:
      requests:
        memory: 256Mi
        cpu: 500m
  controller:
    resources:
      requests:
        memory: 256Mi
        cpu: 500m
clair:
  clair:
    resources:
      requests:
        memory: 256Mi
        cpu: 500m
  adapter:
    resources:
      requests:
        memory: 256Mi
        cpu: 500m
notary:
  server:
    resources:
      requests:
        memory: 256Mi
        cpu: 500m
  signer:
    resources:
      requests:
        memory: 256Mi
        cpu: 500m
database:
  internal:
    resources:
      requests:
        memory: 256Mi
        cpu: 500m
redis:
  internal:
    resources:
      requests:
        memory: 256Mi
        cpu: 500m
trivy:
  enabled: true
  resources:
    requests:
      cpu: 200m
      memory: 512Mi
    limits:
      cpu: 1000m
      memory: 1024Mi

#开启 chartmuseum，使 Harbor 能够存储 Helm 的 chart
chartmuseum:
  enabled: true
  resources:
    requests:
     memory: 256Mi
     cpu: 500m     
```



### 客户端使用

#### 配置host(无公网域名)

客户端想通过域名访问服务，必须要进行 DNS 解析，由于这里没有 DNS 服务器进行域名解析，所以修改 hosts 文件将 Harbor 指定节点的 IP 和自定义 host 绑定。打开电脑的 Hosts 配置文件，往其加入下面配置：

```bash
192.168.2.11  harbor.example.io
```

访问 https://harbor.example.io

```
用户：harbor
密码：Harbor123456 (在安装配置中自定义的密码)
```

#### 配置镜像仓库

在上面部署 harbor 过程中创建了 https 证书 ca.crt。

##### docker 配置证书

在服务器上 /etc/docker 目录下创建 certs.d 文件夹，然后在 certs.d 文件夹下创建 Harobr 域名文件夹，可以输入下面命令创建对应文件夹：

```
mkdir -p /etc/docker/certs.d/harbor.example.io
```

然后再 `/etc/docker/certs.d/harbor.example.io `目录下创建上面的 ca 证书文件：

##### 登录Harbor仓库

只有登录成功后才能将镜像推送到镜像仓库，所以配置完证书后尝试登录，测试是否能够登录成功：

```
docker login -u admin -p Harbor123456 harbor.example.io
```

执行更新,使证书生效

```
update-ca-trust extract
```



### 功能测试

#### 推送拉取镜像

```bash
# 拉取测试镜像
docker pull hell-world:latest

# 使用tag改变镜像
docker tag hello-world:latest horbor.example.io/library/hello-world:latest

# 推送
docker push harbor.example.io/library/hello-world:latest
```



#### 推送拉取Chart

```bash
# 安装插件
helm plugin install https://github.com/chartmuseum/helm-push

# 添加helm repo 
helm repo add local-repo https://harbor.example.io/charts/library --username=admin --password=Harbor123456

# 创建测试chart
helm create hello

# 打包
helm package hello

# 推送
helm push helo-0.0.1.tgz local-repo
```



#### 镜像缓存

```yaml
# 配置镜像地址
# docker
# /etc/docker/daemon.json
{

}

# containerd(k3s)
# /etc/rancher/k3s/registries.yaml
mirrors:
  "docker.io":
    endpoint:
      - "https://harbor.example.io"
  "harbor.example.io":
    endpoint:
      - "https://harbor.example.io"

```



---



### 使用ClusterIP部署

```yaml
expose:
  type: ClusterIP
  tls:
    enable: true
    certSource: secret
    secret:
      secretName: harbor-tls
      notarySecretName: harbor-tls
externalURL: https://harbor.example.io
presentation:
  storageClass: local-path
```

#### 使用Ingress部署

```yaml
expose:
  type: Ingress
  tls:
    enable: true
    certSource: secret
  secret:
    secretName: harbor-tls
    notarySecretName: harbor-tls
  ingress:
    hosts: 
      core: harbor.example.io
      notary: notary.example.io
    controller: default
    annotations:
      ingress.kubernetes.io/ssl-redirect: "false"
      kubernetes.io/ingress.class: "traefik"
      ingress.kubernetes.io/proxy-body-size: "0"
      nginx.ingress.kubernetes.io/ssl-redirect: "true"
      nginx.ingress.kubernetes.io/proxy-body-size: "0"
    notary:
      annotations: {}
    harbor:
      annotations: {}
    
    
    
```



[Kubernetes1.21搭建harbor-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1872039)



traefik 对外暴露应用

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  namespace: kube-ops
  name: harbor-http
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`harbor.xxx.com`) && PathPrefix(`/`)
      kind: Rule
      services:
        - name: harbor-portal
          port: 80
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  namespace: kube-ops
  name: harbor-api
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`harbor.xxx.com`) && PathPrefix(`/api`)
      kind: Rule
      services:
        - name: harbor-core
          port: 80
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  namespace: kube-ops
  name: harbor-service
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`harbor.xxx.com`) && PathPrefix(`/service`)
      kind: Rule
      services:
        - name: harbor-core
          port: 80
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  namespace: kube-ops
  name: harbor-v2
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`harbor.xxx.com`) && PathPrefix(`/v2`)
      kind: Rule
      services:
        - name: harbor-core
          port: 80
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  namespace: kube-ops
  name: harbor-chartrepo
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`harbor.xxx.com`) && PathPrefix(`/chartrepo`)
      kind: Rule
      services:
        - name: harbor-core
          port: 80
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  namespace: kube-ops
  name: harbor-c
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`harbor.xxx.com`) && PathPrefix(`/c`)
      kind: Rule
      services:
        - name: harbor-core
          port: 80
```

