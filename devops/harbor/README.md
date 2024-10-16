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

##### 使用LocalPath(K3s)

```bash
kubectl apply -f harbor-pvc.yaml
```


#### 创建证书

##### 使用cert-manager生成证书

参考[cert-manager生成证书](../cert-manager/README.md)

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
# 使用ClusterIP 文件改为 harbor-values-with-cluster-ip.yaml
helm upgrade harbor harbor/harbor \
	--namespace harbor --install \
	--create-namespace -f harbor-values-with-ingress.yaml  
	
# 查看部署是否完成
kubectl get deployment -n harbor
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
    "registry-mirrors":[],
    "insecure-registries":[https://harbor.example.io]
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





