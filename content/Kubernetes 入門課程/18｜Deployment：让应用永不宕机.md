# Deployment：讓應用永不宕機

## 📝 課程概述

Deployment 是 Kubernetes 最核心的 API 對象之一，專門用於部署無狀態的在線業務應用。透過 replicas 和 selector 的組合設計，實現應用的多實例運行、自動故障恢復，達成「永不宕機」的目標。

## 核心觀念與實作解析

### 為什麼需要 Deployment

Pod 本身無法管理自己，存在以下問題：

- **restartPolicy** 只能保證容器正常工作，但 Pod 本身出錯（如誤刪、節點故障）時無法恢復
- 在線業務需求複雜：多實例、高可用、版本更新等
- 手工管理多個 Pod 副本無法利用 Kubernetes 自動化運維優勢

解決方案：採用「對象套對象」的設計模式，讓 Deployment 管理 Pod，實現與 Job/CronJob 相似的控制邏輯。

### Deployment 的 YAML 結構

使用 `kubectl create` 生成樣板：

```bash
export out="--dry-run=client -o yaml"
kubectl create deploy ngx-dep --image=nginx:alpine $out
```

生成的 YAML 包含三個關鍵部分：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: ngx-dep
  name: ngx-dep

spec:
  replicas: 2          # 關鍵字段 1：副本數量
  selector:            # 關鍵字段 2：標籤選擇器
    matchLabels:
      app: ngx-dep

  template:            # 關鍵字段 3：Pod 模板
    metadata:
      labels:
        app: ngx-dep
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
```

### replicas：副本數量管理

`replicas` 字段定義了 Pod 的「期望數量」，Kubernetes 自動維護：

- Deployment 剛創建時，Pod 數量為 0，會根據模板逐個創建
- 若 Pod 因故障消失，會自動選擇新節點創建補充
- 持續監控，確保 Pod 數量與期望狀態一致

### selector：標籤選擇機制

為什麼需要 selector？這是因為 Deployment 和 Pod 是**鬆散的組合關係**：

- **離線業務**（Job）：Pod 是一次性的，與 Job 強绑定
- **在線業務**（Deployment）：Pod 永遠在線，可能被多個對象引用（如 Service）

Kubernetes 採用「貼標籤」設計：

- 在 `metadata.labels` 添加標籤
- `selector.matchLabels` 定義篩選規則
- 解除強绑定，形成「弱引用」關係

**重要**：`selector.matchLabels` 必須與 `template.metadata.labels` 完全一致，否則 apiserver 會報錯。

### Deployment 的組合關係圖解

```
┌─────────────────────────────────────────────┐
│                 Deployment                   │
│  ┌─────────────┐    ┌───────────────────┐   │
│  │  replicas   │    │     selector      │   │
│  │    = 2      │    │  matchLabels:     │   │
│  └─────────────┘    │    app: ngx-dep ──┼───┼─┐
│                     └───────────────────┘   │ │
│  ┌───────────────────────────────────────┐ │ │
│  │              template                 │ │ │
│  │  ┌─────────────────────────────────┐ │ │ │
│  │  │         Pod 模板                 │ │ │ │
│  │  │  labels:                        │ │ │ │
│  │  │    app: ngx-dep ────────────────┼─┼─┼─┘
│  │  │  containers:                    │ │ │
│  │  │    - nginx:alpine               │ │ │
│  │  └─────────────────────────────────┘ │ │
│  └───────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
              │
              ▼  (透過 selector 篩選管理)
    ┌─────────────────┐  ┌─────────────────┐
    │    Pod #1       │  │    Pod #2       │
    │ ngx-dep-xxx-aaa │  │ ngx-dep-xxx-bbb │
    │ labels:         │  │ labels:         │
    │   app: ngx-dep  │  │   app: ngx-dep  │
    └─────────────────┘  └─────────────────┘
```

### kubectl 操作 Deployment

#### 查看狀態

```bash
kubectl get deploy
```

輸出字段含義：
- **READY**：當前運行數/期望數（如 `2/2`）
- **UP-TO-DATE**：已更新到最新狀態的 Pod 數量
- **AVAILABLE**：健康且可對外服務的 Pod 數量（最重要指標）
- **AGE**：運行時間

```bash
kubectl get pod
```

Pod 命名規則：`Deployment 名稱 + 隨機 Hash 值`

#### 验證自動恢復

```bash
# 刪除一個 Pod 模擬故障
kubectl delete pod ngx-dep-xxx-aaa

# 再次查看，會發現新 Pod 已自動創建
kubectl get pod
```

#### 應用伸缩

```bash
# 扩容到 5 个副本
kubectl scale --replicas=5 deploy ngx-dep

# 編輯 YAML 以声明式修改（推薦）
# 修改 replicas 字段後執行
kubectl apply -f deploy.yml
```

#### 使用 labels 查询

```bash
# 查找特定標籤的 Pod
kubectl get pod -l app=nginx

# 使用 in 表達式
kubectl get pod -l 'app in (ngx, nginx, ngx-dep)'

# 查看所有標籤
kubectl get pod --show-labels
```

## 💡 重點摘要

- Deployment 是管理 Pod 的「外殼」，實現應用永不宕機的自動維護
- `replicas` 定義期望副本數，Kubernetes 自動調整 Pod 敦量
- `selector` 透過 labels 篩選 Pod，形成鬆散的組合關係
- Deployment 和 Pod 的 labels 必須完全一致才能成功创建
- 即使只需運行一個 Pod，也應使用 Deployment（replicas=1）
- Deployment 適用於**無狀態應用**，有狀態應用需用 StatefulSet

## 🔑 關鍵字

Deployment, replicas, selector, labels, matchLabels, 无状态应用