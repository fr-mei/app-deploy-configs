# 应用部署配置仓库 (app-deploy-configs)

这个仓库包含Kubernetes应用的配置清单，用于与GitOps工作流程（如Argo CD）集成。

## 目录结构

```
app-deploy-configs/
├── apps/                      # 应用配置目录
│   ├── production/            # 生产环境配置
│   │   └── go-api/            # Go API应用
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       ├── configmap.yaml
│   │       └── kustomization.yaml
│   └── staging/              # 预发环境配置
├── infrastructure/           # 基础设施配置
│   ├── bootstrap/           # ArgoCD和基础服务配置
│   │   ├── tcr-secret.yaml        # TCR镜像仓库访问凭据
│   │   ├── argocd-repo-config.yaml # GitHub仓库访问配置
│   │   ├── go-api-app.yaml        # go-api应用定义
│   │   └── kustomization.yaml     # 基础设施资源清单
│   └── bootstrap-app.yaml   # 引导应用定义
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

### 应用部署流程
1. 开发人员编写应用代码并推送到应用源码仓库
2. CI管道构建应用镜像并推送到容器镜像仓库
3. CI管道自动更新 `apps/` 目录中的配置清单，引用新的镜像版本
4. Argo CD检测到配置变更，自动将应用部署到Kubernetes集群

### 基础设施部署流程
1. 管理员修改 `infrastructure/` 目录中的基础设施配置
2. 提交并推送更改到此仓库
3. Argo CD自动同步基础设施配置到集群

### 初始化步骤
1. K8s集群搭建完成并安装ArgoCD后，执行以下命令进行一次性引导：
   ```bash
   # 将引导应用文件传输到主节点
   scp -i <SSH_KEY_PATH> infrastructure/bootstrap-app.yaml <SSH_USER>@<MASTER_IP>:~/
   
   # 登录主节点并应用引导配置
   ssh -i <SSH_KEY_PATH> <SSH_USER>@<MASTER_IP>
   kubectl apply -f ~/bootstrap-app.yaml
   ```

2. 后续所有配置更改只需提交到Git仓库，ArgoCD会自动同步

## 最佳实践

- 使用版本标签而非latest标签引用容器镜像
- 使用kustomization.yaml组织相关资源
- 将敏感信息存储在Kubernetes Secrets中，不要直接提交到Git仓库
- 为不同环境创建独立的目录（如production、staging）

## go-api

### 1. 通过Service的ClusterIP访问（集群内部）
```
# 获取Service的ClusterIP
kubectl get svc go-api -n default

# 使用ClusterIP访问
curl http://<cluster-ip>:80/api/endpoint
```
### 2. 通过NodePort访问（外部访问）
您的Service类型为LoadBalancer，腾讯云会自动创建公网负载均衡器。获取公网IP：
```
kubectl get svc go-api -n default -o wide
```
在输出中找到EXTERNAL-IP字段，使用该IP访问：
```
curl http://<external-ip>:80/api/endpoint
```
### 3. 通过Ingress访问（推荐）
如果配置了Ingress，可以使用域名访问：

```bash
curl http://your-domain.com/api/endpoint
```
### 4. 端口转发临时访问
```bash
kubectl port-forward svc/go-api 8080:80
```
在本地浏览器访问：http://localhost:8080
