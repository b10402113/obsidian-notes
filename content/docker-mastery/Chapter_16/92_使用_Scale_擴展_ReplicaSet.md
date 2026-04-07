# 使用 Scale 擴展 ReplicaSet

## 📝 課程概述

本單元學習如何使用 `kubectl scale` 指令擴展 Deployment 的 Pod 數量。我們將理解 Kubernetes 的聲明式設計如何影響擴縮容行為，以及什麼情況下會觸發新 ReplicaSet 的建立。

---

## 核心觀念與實作解析

### 建立 Apache Deployment

這次使用 Apache（httpd）作為範例，因為它會產生較多的 log 資訊，方便後續觀察：

```bash
kubectl create deployment my-apache --image httpd
```

> Docker Hub 上的官方 image 稱為 `httpd`，這是 Apache server 的 daemon 名稱。

#### 查看建立的資源

```bash
kubectl get all
```

輸出會顯示：
- **Deployment**：`my-apache` — 1/1 ready
- **ReplicaSet**：`my-apache-{hash}` — 1/1/1
- **Pod**：`my-apache-{hash}-{id}` — Running

三個層級的狀態數字一致，因為它們在毫秒級時間內連鎖建立完成。

---

### 擴展 Deployment

```bash
kubectl scale deploy/my-apache --replicas 2
```

#### 資源名稱的簡寫規則

Kubernetes 支援多種資源名稱寫法：

```bash
# 以下寫法皆等效
kubectl scale deployment my-apache --replicas 2
kubectl scale deployments my-apache --replicas 2
kubectl scale deploy my-apache --replicas 2
kubectl scale deploy/my-apache --replicas 2
```

> 實務上常用 `deploy` 簡寫搭配 `/` 分隔符號，簡潔明確。

#### 擴展後的狀態

```bash
kubectl get all
```

現在會看到：
- Deployment：2/2 ready
- ReplicaSet：2/2/2
- **兩個 Pod**：名稱後綴 ID 不同

---

### Scale 指令的聲明式語意

**`--replicas` 參數是「目標數量」，而非「增量」。**

```bash
# 如果原本有 10 個 Pod
kubectl scale deploy/my-apache --replicas 2
# 結果：終止 8 個 Pod，保留 2 個

# 如果原本有 1 個 Pod
kubectl scale deploy/my-apache --replicas 2
# 結果：新增 1 個 Pod，總共 2 個
```

> 這是 Kubernetes **聲明式 API** 的核心精神：你指定「最終狀態」，系統決定「如何達成」。

---

### Scale 不會建立新的 ReplicaSet

這是一個重要的區別：

| 操作 | 是否建立新 ReplicaSet | 原因 |
| --- | --- | --- |
| **增加/減少 replicas** | 否 | Pod spec 未變，只是數量調整 |
| **更換 image tag** | 是 | Pod spec 改變，需要新版本 |
| **修改環境變數** | 是 | Pod spec 改變，需要新版本 |

當 Scale 改變 replicas 數量時：

1. Controller Manager 偵測到 Deployment 的 replica 欄位變更
2. **直接修改現有 ReplicaSet** 的 replica 數量
3. ReplicaSet Controller 根據新數量建立/刪除 Pod
4. Scheduler 分配新 Pod 到適當的 node
5. Kubelet 啟動新容器

> 只有當 **Pod spec 本身改變** 時，才會建立新的 ReplicaSet，這是滾動更新的基礎。

---

### 擴展流程圖解

```
kubectl scale --replicas 2
         ↓
    Deployment spec 更新（replicas: 2）
         ↓
    ReplicaSet spec 更新（replicas: 2）
         ↓
    ReplicaSet Controller 偵測差異
         ↓
    建立新 Pod（若需要增加）
         ↓
    Scheduler 分配 node
         ↓
    Kubelet 啟動容器
```

這整個流程通常在幾秒內完成。

---

### 為什麼多 Pod 能提升可用性？

當我們將 replicas 設為 2 時：

- **負載分散**：請求可分配到不同 Pod
- **容錯能力**：一個 Pod 故障時，另一個仍可服務
- **滾動更新**：更新時可逐步替換，保持服務不中斷

> 在單一 node 環境中，這相當於執行了兩次 `docker run httpd`。

---

## 💡 重點摘要

- **`kubectl scale` 使用聲明式語意，指定的是「最終 Pod 數量」而非增量。**
- **資源類型支援多種簡寫（deployment/deploy/deployments），實務上常用 `deploy/name` 格式。**
- **單純調整 replicas 不會建立新 ReplicaSet，只有 Pod spec 改變才會。**
- **多 replicas 提供負載分散、容錯能力與滾動更新支援。**
- **Scale 流程由 Controller Manager → ReplicaSet Controller → Scheduler → Kubelet 連鎖觸發。**

---

## 🔑 關鍵字

Scale, Replicas, ReplicaSet, Declarative API, Pod Spec, Rolling Update, Controller Manager
