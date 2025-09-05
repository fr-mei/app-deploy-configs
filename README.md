# 应用部署配置仓库 (app-deploy-configs)

这个仓库包含Kubernetes应用的配置清单，用于与GitOps工作流程（如Argo CD）集成。

## 目录结构

```
app-deploy-configs/
├── apps/                      # 应用配置目录
│   ├── production/            # 生产环境配置
│   │   └── nginx/             # Nginx应用
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       └── kustomization.yaml
│   └── staging/              # 预发环境配置
└── common/                   # 共享配置和资源
    └── namespaces/           # 命名空间定义
```

## 使用方法

1. 将此仓库克隆到本地进行修改：
   ```bash
   git clone https://github.com/your-org/app-deploy-configs.git
   ```

2. 创建或修改应用配置后提交并推送：
   ```bash
   git add .
   git commit -m "更新Nginx配置"
   git push
   ```

3. Argo CD会自动同步更改到Kubernetes集群。

## GitOps工作流

1. 开发人员编写应用代码并推送到应用源码仓库
2. CI管道构建应用镜像并推送到容器镜像仓库
3. 开发人员更新此仓库中的配置清单，引用新的镜像版本
4. Argo CD检测到配置变更，自动将应用部署到Kubernetes集群

## 最佳实践

- 使用版本标签而非latest标签引用容器镜像
- 使用kustomization.yaml组织相关资源
- 将敏感信息存储在Kubernetes Secrets中，不要直接提交到Git仓库
- 为不同环境创建独立的目录（如production、staging）
