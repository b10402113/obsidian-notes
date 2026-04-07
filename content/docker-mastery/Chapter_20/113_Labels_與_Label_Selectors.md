# Labels 與 Label Selectors

## 📝 課程概述

本單元深入探討 Kubernetes 中至關重要的 metadata：**Labels** 與 **Annotations**。這兩者看似簡單，卻是 Kubernetes 資源之間「互相找到彼此」的關鍵機制。我們將學習 Service 如何透過 Label Selector 找到對應的 Pod，以及如何使用 labels 進行資源的篩選與管理。

---

## 核心觀念與實作解析

### Labels 是什麼？

Labels 是附加在 Kubernetes 資源上的 key-value pairs，用於**描述資源的屬性**：

```yaml
metadata:
  labels:
    app: nginx
    environment: production
    tier: frontend
```

#### Labels 的特性

- **可選的**：不是每個資源都必須有 labels
- **彈性的**：key 和 value 可以自訂，沒有嚴格的標準
- **有限的**：有長度與字元的限制

#### 常見的 Label 用途

| Label | 說明 |
|-------|------|
| `app` | 應用程式名稱 |
| `environment` | 環境類型（dev/test/prod） |
| `tier` | 架構層級（frontend/backend） |
| `client` | 客戶名稱（多用於 SaaS 場景） |

---

### Annotations 是什麼？

Annotations 與 Labels 類似，但用途不同：

| 特性 | Labels | Annotations |
|------|--------|-------------|
| **用途** | 描述資源、篩選、關聯 | 儲存配置資料 |
| **大小限制** | 嚴格 | 較寬鬆 |
| **典型場景** | Service 找 Pod | Ingress 設定、自訂配置 |

Annotations 通常用於：

- Ingress controller 的配置
- 第三方工具的自訂設定
- 非用於篩選的元資料

---

### 使用 Labels 篩選資源

#### 從命令列篩選

```bash
# 列出特定 label 的 Pod
kubectl get pods -l app=nginx

# 多條件篩選
kubectl get pods -l app=nginx,environment=production
```

#### 語法說明

Label selector 支援豐富的語法：

- `app=nginx`：等於
- `environment!=production`：不等於
- `tier in (frontend,backend)`：在集合中

> 老師建議：不需要背誦所有語法，遇到實際需求時再查閱官方文件即可。

---

### Label Selectors：資源關聯的核心機制

這是 Kubernetes 中最重要的概念之一。**Service 如何知道要將流量導向哪些 Pod？答案就是 Label Selectors。**

#### Service 與 Pod 的關聯

```yaml
# Service 定義
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: nginx          # 找所有 label 為 app=nginx 的 Pod
  ports:
    - port: 80
---
# Deployment 定義
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  selector:
    matchLabels:
      app: nginx        # 這個 Deployment 管理的 Pod
  template:
    metadata:
      labels:
        app: nginx      # Pod 的 label（必須與 selector 匹配）
    spec:
      containers:
        - name: nginx
          image: nginx
```

#### 關鍵對應關係

1. **Service selector** → 尋找 Pod
2. **Deployment selector.matchLabels** → 識別要管理的 Pod
3. **Pod template labels** → Pod 實際的 label

> **這三者必須一致**，否則 Service 找不到 Pod，Deployment 也無法正確管理 Pod。

---

### Kubernetes 的保護機制

在新版本的 Kubernetes 中，如果 selector 與 template labels 不匹配，部署會直接失敗：

```
Error: selector does not match template labels
```

這是為了防止你意外建立無法被管理的資源。

#### 舊版本的差異

在舊版本中，Kubernetes 會自動填補缺漏的 labels，但這可能導致混亂。現在強制要求明確定義，是更好的做法。

---

### Label Selectors 的進階應用

#### 控制資源排程位置

Labels 不只用於資源關聯，還可以控制 Pod 被排程到哪些 Node：

```bash
# 為 Node 加上 label
kubectl label node node-1 zone=dmz

# 在 YAML 中指定 node selector
spec:
  nodeSelector:
    zone: dmz
```

#### Taints 與 Tolerations

這是更進階的排程控制機制：

- **Taints**：標記 Node「不歡迎」某些 Pod
- **Tolerations**：讓 Pod「可以容忍」某些 Taints

> 老師建議：除非有特殊需求，否則先保持簡單，避免過度複雜化。

---

### Labels 與 Swarm 的比較

| 特性 | Kubernetes | Docker Swarm |
|------|------------|--------------|
| 可見性 | 完全透明，需手動設定 | 自動處理，較不透明 |
| 彈性 | 高度可自訂 | 較受限 |
| 複雜度 | 較高 | 較低 |

Kubernetes 的方式雖然需要更多設定，但提供了更強大的控制能力。

---

## 💡 重點摘要

- **Labels 是 Kubernetes 資源關聯的核心，Service 透過 Label Selectors 找到對應的 Pod。**
- **Service selector、Deployment selector、Pod labels 三者必須一致，否則無法正確運作。**
- **Annotations 用於儲存配置資料，不用於篩選或關聯。**
- **Label Selectors 也可用於控制 Pod 的排程位置，實現進階的資源管理。**

---

## 🔑 關鍵字

Labels, Label Selectors, Annotations, matchLabels, nodeSelector
