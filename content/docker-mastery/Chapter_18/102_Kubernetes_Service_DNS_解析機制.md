# Kubernetes Service DNS 解析機制

## 📝 課程概述

本單元深入探討 Kubernetes 的 DNS 運作機制。我們將理解 Fully Qualified Domain Name（FQDN）的組成結構，以及 Namespace 如何影響 DNS 解析。這些知識對於跨 Namespace 的服務通訊至關重要。

---

## 核心觀念與實作解析

### DNS 在 Kubernetes 中的角色

DNS 在 Kubernetes 中是**可選的附加元件**，但實際上每個叢集都會安裝：

- **Kubernetes 1.11 之前**：KubeDNS（已棄用）
- **Kubernetes 1.11 之後**：CoreDNS（現為標準）

> CoreDNS 提供 DNS-based service discovery，讓服務能透過名稱互相發現。

---

### Fully Qualified Domain Name（FQDN）結構

當你在容器內使用 `curl httpenv` 時，實際上是在查詢完整的 FQDN：

```
httpenv.default.svc.cluster.local
│       │       │       │
│       │       │       └── 叢集 DNS 網域名稱
│       │       └── 資源類型（Service）
│       └── Namespace 名稱
└── Service 名稱（hostname）
```

#### 各部分說明

| 部分 | 說明 |
| --- | --- |
| **httpenv** | Service 名稱（hostname） |
| **default** | Namespace 名稱 |
| **svc** | 資源類型（Service） |
| **cluster.local** | 叢集預設網域名稱 |

---

### Namespace 概念

Namespace 是 Kubernetes 的**組織隔離機制**：

```bash
kubectl get namespaces
```

輸出範例：
```
NAME              STATUS   AGE
default           Active   10d
docker            Active   10d    # Docker Desktop 特有
kube-node-lease   Active   10d
kube-public       Active   10d
kube-system       Active   10d
```

#### 預設 Namespace

| Namespace | 用途 |
| --- | --- |
| **default** | 應用程式預設運行的 namespace |
| **kube-system** | Control Plane 元件運行於此 |
| **kube-public** | 公開資訊（很少使用） |
| **kube-node-lease** | Node 心跳資訊（新版本） |

#### Namespace 的特性

- **同一名稱空間內**：資源名稱不可重複
- **不同名稱空間**：可以有同名資源，互不衝突
- **DNS 隔離**：每個 namespace 有獨立的 DNS 空間

> Namespace 與 Swarm Stack 概念相似但不完全相同。Namespace 是組織工具，預設不提供網路隔離。

---

### DNS 解析規則

#### 同一 Namespace 內

只需使用 hostname：

```bash
curl httpenv
# 等同於
curl httpenv.default.svc.cluster.local
```

#### 跨 Namespace 存取

必須使用完整名稱：

```bash
# 從 namespace-a 存取 namespace-b 的服務
curl httpenv.namespace-b.svc.cluster.local
```

或至少指定 namespace：

```bash
curl httpenv.namespace-b
```

---

### 叢集網域名稱

`cluster.local` 是預設的叢集 DNS 網域名稱：

- 使用 kubeadm 建立叢集時可以自訂
- 多叢集環境可能需要修改以避免衝突
- 這是**叢集內部 DNS**，外部服務無法解析

> 如果你有外部 DNS 需求，需要額外設定 Ingress 或 ExternalDNS 等方案。

---

### Namespace 與 DNS 的關係

```
┌─────────────────────────────────────────────────────────┐
│                    cluster.local                        │
│  ┌─────────────────┐      ┌─────────────────┐          │
│  │    default      │      │   namespace-b   │          │
│  │                 │      │                 │          │
│  │  httpenv ───────┼──────┼→ httpenv        │          │
│  │  (Service)      │      │  (Service)      │          │
│  │                 │      │                 │          │
│  │  DNS: httpenv   │      │  DNS: httpenv   │          │
│  │       .default  │      │       .namespace-b        │
│  └─────────────────┘      └─────────────────┘          │
└─────────────────────────────────────────────────────────┘
```

---

## 💡 重點摘要

- **CoreDNS 是 Kubernetes 的標準 DNS 解決方案，從 1.11 版本開始取代 KubeDNS。**
- **FQDN 格式為 `{hostname}.{namespace}.svc.cluster.local`。**
- **Namespace 提供組織隔離，不同 namespace 可有同名資源。**
- **同一 namespace 內只需使用 hostname；跨 namespace 需指定完整名稱。**
- **`cluster.local` 是預設叢集網域名稱，可在建立叢集時自訂。**

---

## 🔑 關鍵字

CoreDNS, FQDN, Namespace, cluster.local, Service Discovery, hostname
