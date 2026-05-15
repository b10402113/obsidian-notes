# 實戰演练：玩轉 Kubernetes（2）

## 📝 課程概述

本課程是「中級篇」的收尾，透過搭建 WordPress 网站實戰演练，綜合應用 Deployment、Service、Ingress 等中級篇學習的核心 API 對象，實現横向扩容、服務发现和七層負載均衡，提升網站的稳定性和可用性。

## 核心觀念與實作解析

### 中級篇知識要点回顾

#### kubeadm 搭建集群（第 17 談）

- Kubernetes 是云原生时代的操作系统，管理計算資源「池化」
- kubeadm 使用容器技術封装 Kubernetes 组件
- `kubeadm init`、`kubeadm join` 一键搭建生產級集群

#### Deployment（第 18 談）

- 管理 Pod 的在线業務对象
- **replicas**：指定實例數量，支持横向扩容/缩容
- **selector**：使用 labels 篩選 Pod，实现 API 对象鬆耦合

#### DaemonSet（第 19 談）

- 在每個節點上運行一個 Pod（類似守護進程）
- 適合日志、監控等系统級應用
- **taint/toleration**：控制 Pod 部署策略

#### Service（第 20 談）

- 抽象 Pod IP 地址，提供固定 IP
- iptables 規則負載均衡到後端 Pod
- kube-proxy 实时维护 Pod 状态
- DNS 插件支持域名访问

#### Ingress（第 21 談）

- 七層（L7）負載均衡，基于 HTTP 协议
- Ingress Controller 应用路由規則（如 Nginx）
- Ingress Class 解耦 Ingress 和 Controller
- 通过 NodePort 或 LoadBalancer 对外暴露

### WordPress 网站架構

相比 Docker/minikube 版本的改進：

```
┌─────────────────────────────────────────────────────────────┐
│                      外部用戶                                │
│              http://wp.test 或 NodePort:30088               │
└─────────────────────────────────────────────────────────────┘
                          │
          ┌───────────────┴───────────────┐
          │                               │
          ▼                               ▼
┌───────────────────┐           ┌─────────────────────────────┐
│  NodePort 方式    │           │    Ingress Controller       │
│  wp-svc:30088     │           │    (hostNetwork: true)      │
│  WordPress 直接   │           │    wp-kic-dep               │
└───────────────────┘           └─────────────────────────────┘
                                          │
                                          │  wp.test → wp-svc
                                          ▼
                          ┌─────────────────────────────┐
                          │      Ingress + Class        │
                          │      wp-ing / wp-ink        │
                          └─────────────────────────────┘
                                          │
                                          ▼
                          ┌─────────────────────────────┐
                          │        Service              │
                          │   wp-svc (ClusterIP)        │
                          └─────────────────────────────┘
                                          │
                    ┌─────────────────────┴─────────────────────┐
                    ▼                                           ▼
        ┌───────────────────────┐           ┌───────────────────────┐
        │   WordPress Pod #1    │           │   WordPress Pod #2    │
        │   (Deployment,        │           │   (Deployment,        │
        │    replicas: 2)       │           │    replicas: 2)       │
        │   wordpress:5         │           │   wordpress:5         │
        └───────────────────────┘           └───────────────────────┘
                    │                                           │
                    └─────────────────────┬─────────────────────┘
                                          │  maria-svc (DNS 域名)
                                          ▼
                          ┌─────────────────────────────┐
                          │        Service              │
                          │   maria-svc (ClusterIP)     │
                          └─────────────────────────────┘
                                          │
                                          ▼
                          ┌─────────────────────────────┐
                          │      MariaDB Pod            │
                          │   (Deployment, replicas: 1) │
                          │   mariadb:10                │
                          └─────────────────────────────┘
```

關键改進：

1. **完全舍弃 Docker**：所有应用在 Kubernetes 集群运行
2. **使用 Deployment**：不再使用裸 Pod，稳定性大幅提升
3. **WordPress 横向扩容**：從 1 個實例變成 2 個（可任意扩容）
4. **Service 服務发现**：使用域名访问，无需手动查看 Pod IP
5. **Ingress Controller**：替代原 Nginx 反向代理
6. **兩種对外暴露方式**：
   - WordPress Service NodePort（30088）：方便测试
   - Ingress Controller hostNetwork：避开 NodePort 端口限制

### 部署步驟

#### 步驟 1：部署 MariaDB

**ConfigMap（環境变量）**

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

**Deployment（套外壳）**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: maria-dep

spec:
  replicas: 1
  selector:
    matchLabels:
      app: maria-dep

  template:
    metadata:
      labels:
        app: maria-dep
    spec:
      containers:
      - image: mariadb:10
        name: mariadb
        ports:
        - containerPort: 3306
        envFrom:
        - prefix: 'MARIADB_'
          configMapRef:
            name: maria-cm
```

**Service（域名访问）**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: maria-svc

spec:
  ports:
  - port: 3306
    protocol: TCP
    targetPort: 3306
  selector:
    app: maria-dep
```

執行：

```bash
kubectl apply -f wp-maria.yml
```

#### 步驟 2：部署 WordPress

**ConfigMap（使用 Service 域名）**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: wp-cm

data:
  HOST: 'maria-svc'    # 关键：使用 Service 域名而非 IP
  USER: 'wp'
  PASSWORD: '123'
  NAME: 'db'
```

**Deployment（横向扩容）**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wp-dep

spec:
  replicas: 2          # 2 個實例
  selector:
    matchLabels:
      app: wp-dep

  template:
    metadata:
      labels:
        app: wp-dep
    spec:
      containers:
      - image: wordpress:5
        name: wordpress
        ports:
        - containerPort: 80
        envFrom:
        - prefix: 'WORDPRESS_DB_'
          configMapRef:
            name: wp-cm
```

**Service（NodePort 对外暴露）**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wp-svc

spec:
  ports:
  - name: http80
    port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30088     # 手工指定端口（30000~32767）
  selector:
    app: wp-dep
  type: NodePort       # NodePort 类型
```

執行：

```bash
kubectl apply -f wp-dep.yml
```

访问：`http://192.168.10.210:30088`

#### 步驟 3：部署 Nginx Ingress Controller

**Ingress Class**

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: wp-ink

spec:
  controller: nginx.org/ingress-controller
```

**Ingress**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wp-ing

spec:
  ingressClassName: wp-ink

  rules:
  - host: wp.test
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wp-svc
            port:
              number: 80
```

**Ingress Controller（hostNetwork）**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wp-kic-dep
  namespace: nginx-ingress

spec:
  replicas: 1
  selector:
    matchLabels:
      app: wp-kic-dep

  template:
    metadata:
      labels:
        app: wp-kic-dep
    spec:
      serviceAccountName: nginx-ingress
      hostNetwork: true    # 关键：使用宿主机网络
      containers:
      - image: nginx/nginx-ingress:2.2-alpine
        args:
          - -ingress-class=wp-ink
```

執行：

```bash
kubectl apply -f wp-ing.yml -f wp-kic.yml
```

**域名解析**（修改 `/etc/hosts`）：

```
192.168.10.210   wp.test
```

访问：`http://wp.test`

### 實作技巧

- 善用 `kubectl create`、`kubectl expose` 生成 YAML 样板
- 多個對象可寫在同一 YAML 文件，用 `---` 分隔
- 用 `kubectl get` 確認對象狀態

## 💡 重點摘要

- 中級篇學習：kubeadm、Deployment、DaemonSet、Service、Ingress
- WordPress 架構改進：Deployment 替代裸 Pod、Service 域名访问、Ingress 七層路由
- MariaDB HOST 配置需使用 Service 域名（`maria-svc`）而非 IP
- 兩種對外暴露：NodePort（30088）和 hostNetwork
- 当前仍有问题：MariaDB Pod 重启後数据丢失，需 PersistentVolume 解决

## 🔑 關鍵字

WordPress, Deployment, Service, Ingress, NodePort, hostNetwork, 實戰演练, 中級篇总结