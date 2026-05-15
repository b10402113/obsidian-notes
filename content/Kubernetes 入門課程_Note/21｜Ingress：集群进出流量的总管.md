# Ingress：集群進出流量的總管

## 📝 課程概述

Ingress 是 Kubernetes 在 Service 之上提出的七層負載均衡概念，專門處理 HTTP/HTTPS 協議的流量路由。透過 Ingress、Ingress Controller、Ingress Class 三個對象的協作，實現集群進出流量的統一管理，是微服務架構的關鍵入口。

## 核心觀念與實作解析

### 為什麼需要 Ingress

#### Service 的限制

Service 是四層（L4）負載均衡，只能根據 IP 地址和端口號轉發流量：

- **無法處理 HTTP 層信息**：主機名、URI、請求頭、证书等 TCP/IP 栈看不見
- **對外暴露能力不足**：NodePort、LoadBalancer 缺乏靈活性，難以管控

Ingress 是七層（L7）負載均衡，作為集群流量的「总入口」：

- 使用 HTTP/HTTPS 协议定义路由规则
- 管理「南北向」流量（扇入/扇出）
- 让外部用户安全、顺畅地访问内部服务

```
┌─────────────────────────────────────────────────────────┐
│                        外部用戶                          │
│                   HTTP/HTTPS 请求                        │
└─────────────────────────────────────────────────────────┘
                         │
                         │  七層路由（域名、URI、Header）
                         ▼
┌─────────────────────────────────────────────────────────┐
│                      Ingress Controller                  │
│                   （Nginx/Kong/APISIX 等）               │
└─────────────────────────────────────────────────────────┘
                         │
                         │  四層轉發
                         ▼
┌───────────────┐  ┌───────────────┐  ┌───────────────┐
│   Service A   │  │   Service B   │  │   Service C   │
│  ngx-svc:80   │  │  api-svc:80   │  │  web-svc:80   │
└───────────────┘  └───────────────┘  └───────────────┘
```

### 三個核心對象的關係

#### Ingress vs Service 的對比

Service 本身只是 iptables 規則，真正應用規則的是 **kube-proxy**。

同理，Ingress 只是 HTTP 路由規則集合（靜態描述文件），真正實施規則的是 **Ingress Controller**。

#### 為什麼需要 Ingress Class

最初設計的問題：

- 一個集群只能有一個 Ingress Controller
- Ingress 規則太多會让 Controller 不堪重負
- 多個 Ingress 缺乏邏輯分组
- 不同租户需求冲突，無法部署在同一 Controller

**Ingress Class** 解耦 Ingress 和 Ingress Controller：

- 插在 Ingress 和 Controller 中間
- 作为流量規則和控制器的協調人
- 可定義不同業務邏輯分组（如博客、短视频、購物）

```
┌───────────────────────────────────────────────────────┐
│                    Ingress Class                       │
│                     (ngx-ink)                          │
│  controller: nginx.org/ingress-controller             │
└───────────────────────────────────────────────────────┘
          │                              │
          │  ingressClassName            │  -ingress-class
          ▼                              ▼
┌───────────────────┐          ┌─────────────────────────┐
│     Ingress       │          │   Ingress Controller    │
│    (ngx-ing)      │          │   (ngx-kic-dep)         │
│ rules: ngx.test/  │          │   Deployment/DaemonSet  │
└───────────────────┘          └─────────────────────────┘
```

### Ingress 的 YAML 結構

使用 `kubectl create ing` 生成樣板：

```bash
export out="--dry-run=client -o yaml"
kubectl create ing ngx-ing --rule="ngx.test/=ngx-svc:80" --class=ngx-ink $out
```

生成的 YAML：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ngx-ing

spec:
  ingressClassName: ngx-ink    # 關聯的 Ingress Class

  rules:
  - host: ngx.test             # HTTP 主機名
    http:
      paths:
      - path: /                # URI 路徑
        pathType: Exact        # 匹配方式：Exact/Prefix
        backend:
          service:
            name: ngx-svc      # 轉發目標 Service
            port:
              number: 80
```

關鍵字段：
- **ingressClassName**：指定 Ingress Class
- **rules**：HTTP 路由規則（host + path + backend）
- **pathType**：Exact（精確匹配）或 Prefix（前缀匹配）

### Ingress Class 的 YAML 結構

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: ngx-ink

spec:
  controller: nginx.org/ingress-controller    # Controller 名称
```

### Ingress Controller 的部署

Ingress Controller 本身是 Pod，由 Deployment/DaemonSet 管理。

#### 安装前準備

```bash
# 创建名字空间、账号、权限
kubectl apply -f common/ns-and-sa.yaml
kubectl apply -f rbac/rbac.yaml

# 配置 HTTP/HTTPS
kubectl apply -f common/nginx-config.yaml
kubectl apply -f common/default-server-secret.yaml
```

#### Deployment YAML 关键修改

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ngx-kic-dep
  namespace: nginx-ingress

spec:
  replicas: 1
  selector:
    matchLabels:
      app: ngx-kic-dep

  template:
    metadata:
      labels:
        app: ngx-kic-dep
    spec:
      containers:
      - image: nginx/nginx-ingress:2.2-alpine
        args:
          - -ingress-class=ngx-ink    # 关联 Ingress Class
```

### 对象关联图

```
┌─────────────────────────────────────────────────────────────────┐
│                         外部请求                                 │
│                    curl ngx.test:8080                            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    kubectl port-forward                          │
│                    8080 → Pod:80                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                  Ingress Controller (Pod)                        │
│                  name: ngx-kic-dep                               │
│                  args: -ingress-class=ngx-ink                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │  根据 Ingress 规則
                              ▼
┌───────────────────────────┐     ┌───────────────────────────────┐
│     Ingress Class         │     │          Ingress              │
│     name: ngx-ink         │────▶│     name: ngx-ing             │
│     controller: nginx...  │     │     ingressClassName: ngx-ink │
└───────────────────────────┘     │     rules: ngx.test/          │
                                  │       → ngx-svc:80            │
                                  └───────────────────────────────┘
                                              │
                                              ▼
                                  ┌───────────────────────┐
                                  │      Service          │
                                  │   name: ngx-svc       │
                                  │   ClusterIP: 10.96... │
                                  └───────────────────────┘
                                              │
                                              ▼
                                  ┌───────────────────────┐
                                  │       Pod             │
                                  │   nginx:alpine        │
                                  └───────────────────────┘
```

### 测试验证

使用 `kubectl port-forward` 映射端口：

```bash
kubectl port-forward -n nginx-ingress ngx-kic-dep-xxx 8080:80 &
```

使用域名访问（需指定解析）：

```bash
curl --resolve ngx.test:8080:127.0.0.1 http://ngx.test:8080
```

### Ingress Controller 的扩展功能

現代 Ingress Controller 不只管理入口流量：

- **入口流量**（Ingress）：南北向
- **出口流量**（Egress）
- **東西向流量**：集群内部服务间
- **额外功能**：TLS 终止、WAF、限流限速、流量拆分、身份认证、访问控制

## 💡 重點摘要

- Ingress 是七層（L7）負載均衡，基于 HTTP/HTTPS 协议定义路由規則
- Ingress 只是靜態規則集合，需 Ingress Controller（如 Nginx）實際應用規則
- Ingress Class 解耦 Ingress 和 Controller，支持多租户、多業務分组
- 最流行的 Ingress Controller 是 Nginx Ingress Controller
- Ingress Controller 以 Pod 形式運行，需 Service（NodePort/LoadBalancer）對外暴露

## 🔑 關鍵字

Ingress, Ingress Controller, Ingress Class, 七層負載均衡, HTTP路由, Nginx Ingress Controller