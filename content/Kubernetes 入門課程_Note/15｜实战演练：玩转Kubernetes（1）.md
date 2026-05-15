# Kubernetes 实战演练：搭建 WordPress

## 📝課程概述
本課程综合运用初级篇所学知识，在 Kubernetes 集群里搭建 WordPress 网站，对比 Docker 的部署方式，展示容器编排技术的实际应用。

## 核心觀念與實作解析

### Kubernetes 技術要点回顾

**容器编排的必要性**：
- 容器只针对单个进程的隔离和封装
- 实际应用场景要求多个进程互相协同工作
- 容器编排（Container Orchestration）解决集群级别的调度管理

**Master/Node 架构**：
- Master：apiserver、etcd、scheduler、controller-manager
- Worker：kubelet、kube-proxy、container-runtime
- 常用插件：DNS、Dashboard

**API对象**：
- Pod：捆绑密切协作的容器，共享网络和存储
- Job/CronJob：离线作业，逐层包装 Pod
- ConfigMap/Secret：配置信息，注入 Pod

### WordPress 网站基本架构

与 Docker 版本的架构大体相同，关键区别在于：
1. **应用封装**：WordPress、MariaDB 封装成 Pod
2. **配置管理**：环境变量改用 ConfigMap，声明式管理
3. **网络环境**：Kubernetes 内部维护专用网络，需端口转发

### 搭建步骤

**步骤一：编排 MariaDB**

ConfigMap 定义环境变量：
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: maria-cm
data:
  DATABASE: 'db'
  USER: 'wp'
  PASSWORD: '123'
  ROOT_PASSWORD: '123'
```

Pod 定义（使用 envFrom批量导入）：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: maria-pod
spec:
  containers:
  - image: mariadb:10
    name: maria
    envFrom:
    - prefix: 'MARIADB_'        # 自动添加前缀
      configMapRef:
        name: maria-cm
```

**步骤二：编排 WordPress**

ConfigMap（注意 HOST 字段必须是 MariaDB Pod 的 IP）：
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: wp-cm
data:
  HOST: '172.17.0.2'           # MariaDB Pod IP
  USER: 'wp'
  PASSWORD: '123'
  NAME: 'db'
```

**步骤三：端口映射**

Pod 运行在 Kubernetes 私有网段，需用 `kubectl port-forward`：
```bash
kubectl port-forward wp-pod 8080:80 &
```

**步骤四：Nginx 反向代理**

WordPress 使用 URL 重定向，需 Nginx 反向代理保证端口一致性：
```nginx
server {
    listen 80;
    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

```bash
docker run -d --rm --net=host \
    -v /tmp/proxy.conf:/etc/nginx/conf.d/default.conf \
    nginx:alpine
```

### 使用Dashboard 管理 Kubernetes

```bash
minikube dashboard
```

Dashboard 功能：
- 查看工作负载、Pod 详细信息
- 查看日志、进入 Pod 内部
- 编辑、删除 Pod
- 查看 ConfigMap/Secret

## 💡 重點摘要
- Kubernetes 以「声明式」YAML 描述应用状态和关系
- `envFrom` 可批量导入 ConfigMap，并添加前缀
- Pod IP 需手工查找填写，缺少服务发现机制
- `kubectl port-forward` 用于临时调试测试
- Dashboard 提供图形化管理界面

## 🔑 關鍵字
port-forward, envFrom, prefix, Dashboard, WordPress, MariaDB