# Service：微服務架構的應對之道

## 📝 課程概述

Service 是 Kubernetes 的核心 API 對象，解決了微服務架構中的「服務發現」問題。透過靜態 IP 地址和 DNS 域名，屏蔽後端 Pod 的动态變化，為客戶端提供稳定的服務入口，實現集群內部的負載均衡。

## 核心觀念與實作解析

### 為什麼需要 Service

Pod 的問題：

- Pod 生命周期短暂，IP 地址动态變化
- Deployment/DaemonSet 维持 Pod 數量稳定，但 Pod 銷毀重建後 IP 會改變
- 微服務架構需要稳定的服務入口，客户端無法追蹤變動的 Pod IP

解决方案：**負載均衡**

Service 的工作原理：

- Kubernetes 分配**靜態 IP 地址**
- 自動管理後端动态變化的 Pod 集合
- 使用 iptables/ipvs 技術，由 kube-proxy 维護轉發規則
- 客户端訪問 Service 的固定 IP，流量被轉發到後端 Pod

```
┌───────────────────────────────────────────────────────┐
│                       Service                          │
│  靜態 IP: 10.96.240.115                                │
│  selector: app=ngx-dep                                 │
│  ports: 80 → targetPort: 80                            │
└───────────────────────────────────────────────────────┘
                    │
                    │  kube-proxy (iptables/ipvs)
                    ▼
    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
    │     Pod      │    │     Pod      │    │     Pod      │
    │ 10.10.0.232  │    │ 10.10.1.86   │    │ 10.10.1.87   │
    │    (動態)    │    │    (動態)    │    │    (動態)    │
    └──────────────┘    └──────────────┘    └──────────────┘
```

### Service 的 YAML 結構

使用 `kubectl expose` 命令生成樣板（注意不是 `kubectl create`）：

```bash
export out="--dry-run=client -o yaml"
kubectl expose deploy ngx-dep --port=80 --target-port=80 $out
```

生成的 YAML：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ngx-svc

spec:
  selector:
    app: ngx-dep      # 篩選要代理的 Pod

  ports:
  - port: 80          # Service 對外端口
    targetPort: 80    # Pod 容器端口
    protocol: TCP
```

關鍵字段：

- **selector**：使用 labels 篩選後端 Pod，與 Deployment/DaemonSet 的 selector 机制相同
- **ports**：定義端口映射，port 是 Service 端口，targetPort 是 Pod 端口

### Service 的特性

#### 虚擬 IP 地址

- Service IP 是「虚地址」，不存在实体
- 只用于轉發流量，無法 ping 通
- 地址段独立於 Pod 地址段（如 Service: 10.96.xx.xx，Pod: 10.10.xx.xx）

#### 自動服務發現

- Service 透過 controller-manager 监控 Pod 變化
- Pod 銷毀重建後，自動更新 Endpoint 列表

```bash
# 查看 Service 代理的 Pod IP
kubectl describe svc ngx-svc

# 查看 Endpoint
kubectl get endpoints ngx-svc
```

#### 負載均衡算法

Service 使用最简单的 **round-robin（轮询）** 算法。

### DNS 域名访问

Kubernetes DNS 插件為 Service 提供域名：

域名格式：
- 完整形式：`<对象名>.<名字空间>.svc.cluster.local`
- 简写形式：`<对象名>.<名字空间>` 或 `<对象名>`

```bash
# 在 Pod 内使用域名访问 Service
curl ngx-svc
curl ngx-svc.default
curl ngx-svc.default.svc.cluster.local
```

#### 名字空间（namespace）

Kubernetes 用 namespace 隔離和分组 API 对象：

```bash
kubectl get ns
```

- **default**：默认名字空间
- **kube-system**：核心组件（apiserver、etcd、coredns 等）

Service 域名包含名字空间，避免域名冲突。

### Service 的类型

`type` 字段定义 Service 的工作模式：

| 类型 | 说明 | 用途 |
|------|------|------|
| ClusterIP（默认） | 集群内部访问 | 内部服務通信 |
| NodePort | 每個節點開端口 | 對外暴露服務 |
| LoadBalancer | 云厂商提供 | 云環境 |
| ExternalName | 外部服務映射 | 引用外部服務 |

#### NodePort 类型

修改 YAML：

```yaml
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30651  # 可選，默認随机分配（30000~32767）
```

效果：

- Service 在集群內繼續使用 ClusterIP
- 在每個節點上開啟一個端口（默認範圍 30000~32767）
- 外部可直接訪問節點 IP + 端口

```
外部客戶端
    │
    │  curl 192.168.10.210:30651
    ▼
┌──────────────────┐    ┌──────────────────┐
│    Master 節點    │    │    Worker 節點    │
│  NodePort:30651  │    │  NodePort:30651  │
└──────────────────┘    └──────────────────┘
           │                     │
           └─────────┬───────────┘
                     ▼
              ┌───────────┐
              │  Service  │
              │ ClusterIP │
              └───────────┘
                     │
                     ▼
              ┌──────────────┐
              │  Pod (動態)   │
              └──────────────┘
```

NodePort 的缺點：

1. **端口数量有限**：默認只有 30000~32767（約 2000 多個）
2. **每節點開端口**：大型集群帶來網路通信成本
3. **暴露節點 IP**：安全風險，需额外反向代理

### 實作驗證

```bash
# 创建 Service
kubectl apply -f svc.yml

# 查看 Service 状态
kubectl get svc

# 查看 Service 代理的 Pod
kubectl describe svc ngx-svc

# 在 Pod 内测试负载均衡
kubectl exec -it <pod-name> -- sh
curl ngx-svc  # 多次访问會轮询不同 Pod

# 删除 Pod 测试自動发现
kubectl delete pod <pod-name>
kubectl describe svc ngx-svc  # Endpoint 自動更新
```

## 💡 重點摘要

- Service 解決「服務發現」問題，為动态 Pod 提供稳定的靜態 IP
- Service IP 是「虚地址」，只用于流量轉發，無法 ping 通
- Service 使用 selector 篩選 Pod，與 Deployment/DaemonSet 共用 labels 机制
- DNS 插件提供域名访问，格式為 `对象名.名字空间`
- 默认类型 ClusterIP 只能在集群內访问，NodePort 可對外暴露服務
- NodePort 在每個節點開端口，缺點是端口数量有限、暴露節點 IP

## 🔑 關鍵字

Service, ClusterIP, NodePort, selector, kube-proxy, DNS域名, 名字空间, 负载均衡