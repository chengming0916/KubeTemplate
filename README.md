

# KubeTemplate

KubeTemplate 是一个基于 Kubernetes 的模板集合，旨在帮助开发者快速部署和配置常用的应用与服务。该模板集合支持多种开发工具、数据库、消息中间件和云原生组件的快速部署，适用于 DevOps 工作流。

## 特性

- **快速部署**：提供常见的 Kubernetes 部署模板，开箱即用。
- **灵活配置**：支持自定义配置，例如存储路径、服务暴露方式等。
- **工具集成**：集成 Drone、Gitea、Harbor、Tekton、SonarQube 等 DevOps 工具。
- **服务部署**：提供 PostgreSQL、Redis、MySQL、EMQX、MinIO 等服务的部署配置。
- **支持多种存储后端**：支持 HostPath、LocalPath 和 NFS 等存储方式。

## 使用场景

- 搭建本地或云端的 DevOps 环境
- 快速部署微服务依赖的数据库与中间件
- 配置自签名或 Let’s Encrypt 证书
- 部署 Kubernetes 生态组件，如 Traefik、ArgoCD、Cert-Manager 等

## 安装与配置

1. **部署 Kubernetes 应用**：
   使用提供的 `deployment.yaml`、`service.yaml`、`pvc.yaml` 等文件部署所需服务。

2. **配置 Ingress**：
   使用 `ingress.yaml` 配置外部访问，结合 Traefik 等 Ingress 控制器。

3. **配置自动补全**：
   - 对于 `kubectl`，请配置相应的自动补全脚本。
   - 对于 `helm`，同样配置自动补全支持。

4. **配置 Traefik**：
   - 根据官方文档配置 K3s 内置的 Traefik。
   - 将配置文件复制到 `/var/lib/rancher/k3s/server/manifests/` 目录下以自动生效。
   - 重启 K3s 服务以加载新配置。

5. **配置自签名证书**：
   使用 `cert-manager` 配置自签名证书和 Let’s Encrypt 证书，确保服务安全访问。

6. **部署 Dashboard**：
   配置并部署 Kubernetes Dashboard，用于图形化管理集群。

7. **部署数据库与中间件**：
   - PostgreSQL、Redis、MySQL、EMQX 等数据库与中间件提供一键部署配置。
   - 可使用 PVC 配置持久化存储，支持 HostPath、LocalPath 和 NFS。

## 参考文档

- [Kubernetes 官方文档](https://kubernetes.io/docs/)
- [K3s 官方文档](https://docs.k3s.io/)
- [Helm 官方文档](https://helm.sh/docs/)
- [Traefik 官方文档](https://doc.traefik.io/traefik/)
- [Cert-Manager 官方文档](https://cert-manager.io/docs/)

## 项目结构

```
├── devops/         # DevOps 工具部署配置
│   ├── argocd/
│   ├── cert-manager/
│   ├── drone/
│   ├── gitea/
│   ├── harbor/
│   ├── tekton/
│   └── sonar/
├── emqx/           # EMQX 部署配置
├── etcd/           # ETCD 部署配置
├── minio/          # MinIO 部署配置
├── monibuca/       # Monibuca 部署配置
├── mysql/          # MySQL 部署配置
├── postgresql/     # PostgreSQL 部署配置
├── redis/          # Redis 部署配置
├── seaweedfs/      # SeaweedFS 部署配置
├── traefik/        # Traefik 配置
├── whoami/         # 测试服务 whoami 部署配置
├── zlmediakit/     # ZLMediaKit 部署配置
├── .gitignore
├── LICENSE
├── README.md
└── registries.yaml # 容器镜像仓库配置
```

## 贡献指南

欢迎提交 Pull Request 和 Issue，为项目提供更完善的部署模板和文档支持。

## 许可证

本项目遵循 MIT License，请查看 [LICENSE](LICENSE) 文件。