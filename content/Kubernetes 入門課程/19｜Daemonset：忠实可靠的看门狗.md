# DaemonSet：忠實可靠的看門狗

## 📝 課程概述

DaemonSet 是 Kubernetes 用於部署「節點守護進程」的 API 對象，會在集群的每個節點上運行且僅運行一個 Pod。與 Deployment 的差異在於調度策略，適用於監控、日誌收集、網路代理等與節點绑定的基礎設施類業務。

## 核心觀念與實作解析

### 為什麼需要 DaemonSet

Deployment 的限制：不關心 Pod 在哪個節點運行，只維護 Pod 數量。但某些業務與節點存在「绑定」關係：

- **網路應用**（如 kube-proxy）：每個節點必須運行一個 Pod，否節點無法加入 Kubernetes 網路
- **監控應用**（如 Prometheus Node Exporter）：每個節點需要 Pod 監控狀態
- **日誌應用**（如 Fluentd）：每個節點需要 Pod 收集容器日誌
- **安全應用**：每個節點需要 Pod 執行安全審計、漏洞掃描

DaemonSet 的目標：**在每個節點上運行且僅運行一個 Pod**，Pod 數量與節點數量同步。

### DaemonSet 的 YAML 結構

DaemonSet 沒有 `kubectl create` 直接生成樣板的命令，可用以下方式創建：

```bash
# 方法 1：從 Deployment 改寫
export out="--dry-run=client -o yaml"
kubectl create deploy redis-ds --image=redis:5-alpine $out
# 然後修改 kind 為 DaemonSet，删除 replicas 字段

# 方法 2：從官網範例抄寫
# https://kubernetes.io/zh/docs/concepts/workloads/controllers/daemonset/
```

YAML 範例：

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: redis-ds
  labels:
    app: redis-ds

spec:
  selector:
    matchLabels:
      name: redis-ds

  template:
    metadata:
      labels:
        name: redis-ds
    spec:
      containers:
      - image: redis:5-alpine
        name: redis
        ports:
        - containerPort: 6379
```

### DaemonSet vs Deployment

| 特性 | DaemonSet | Deployment |
|------|-----------|------------|
| replicas 字段 | **無** | 有 |
| Pod 數量 | 等於節點數量 | 由 replicas 指定 |
| Pod 調度 | 每節點一個 | 不關心節點位置 |
| 用途 | 系統級、節點守護 | 普通業務應用 |

```
┌─────────────────────────────────────────────────────┐
│                    DaemonSet                         │
│  spec:                                              │
│    selector: matchLabels                            │
│    template: Pod 模板（無 replicas）                 │
└─────────────────────────────────────────────────────┘
            │
            ▼  (自動在每個節點創建一個 Pod)
    ┌───────────┐    ┌───────────┐    ┌───────────┐
    │  Node A   │    │  Node B   │    │  Node C   │
    │ Pod #1    │    │ Pod #1    │    │ Pod #1    │
    │ redis-ds  │    │ redis-ds  │    │ redis-ds  │
    └───────────┘    └───────────┘    └───────────┘
```

### 污點（taint）與容忍度（toleration）

Master 節點默認有 taint，拒絕 Pod 調度：

```bash
kubectl describe node master
# Taints: node-role.kubernetes.io/master:NoSchedule
# (或新版本: node-role.kubernetes.io/control-plane:NoSchedule)
```

#### 方法 1：去除節點污點

```bash
# 去除 Master 的污點
kubectl taint node master node-role.kubernetes.io/master:NoSchedule-

# 重新添加污點
kubectl taint node master node-role.kubernetes.io/master:NoSchedule
```

影響：修改 Node 狀態，可能導致所有 Pod 都可調度到此節點。

#### 方法 2：為 Pod 添加容忍度（推薦）

```yaml
spec:
  template:
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
        operator: Exists
```

優點：精細化控制，只讓特定 Pod 運行在有污點的節點。

**注意**：tolerations 是 Pod 的屬性，可在 Job/CronJob、Deployment 中同樣使用。

### 靜態 Pod（Static Pod）

DaemonSet 的替代方案：

- **不受 Kubernetes 系統管控**，不與 apiserver、scheduler 交互
- YAML 文件存放於 `/etc/kubernetes/manifests/` 目錄
- 由節點上的 **kubelet** 直接管理

Kubernetes 核心組件（apiserver、etcd、scheduler、controller-manager）本身就是靜態 Pod。

適用場景：DaemonSet無法滿足的特殊需求，應**慎用**。

## 💡 重點摘要

- DaemonSet 用於在每個節點運行「守護進程」，Pod 數量等於節點數量
- DaemonSet YAML 與 Deployment 類似，**沒有 replicas 字段**
- 「污點」（taint）是 Node 屬性，「容忍度」（toleration）是 Pod 屬性
- Master 節點默認有 taint，Pod 需添加 toleration 才能調度
- 靜態 Pod 是 DaemonSet 的替代方案，由 kubelet 直接管理，不受 Kubernetes 控制

## 🔑 關鍵字

DaemonSet, taint, toleration, 靜態 Pod, kubelet, 節點守護進程