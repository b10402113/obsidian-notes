# Deployment 與 ReplicaSet 的關係

## 📝 課程概述

本單元深入探討 Kubernetes 最核心的資源類型 — **Deployment**。我們將理解 Deployment 如何透過 ReplicaSet 間接管理 Pod，以及這層抽象為何是實現滾動更新的關鍵設計。

---

## 核心觀念與實作解析

### Deployment：最常用的資源類型

Deployment 是 Kubernetes 中**最普及的資源類型**，其地位相當於 Swarm 中的 service。

適用場景：
- Web server（Node.js、PHP、Golang）
- Worker process（Java、Python）
- 無狀態應用程式

> 其他資源類型（StatefulSet、DaemonSet、Job、CronJob）有更特定的用途，Deployment 則是通用型選擇。

---

### 建立 Deployment

```bash
kubectl create deployment my-nginx --image nginx
```

與 `kubectl run` 的關鍵差異：

| 項目 | kubectl run | kubectl create deployment |
| --- | --- | --- |
| 建立的資源 | 單一 Pod | Deployment → ReplicaSet → Pod |
| 生產適用性 | 僅限測試 | 支援滾動更新、擴縮容 |
| 管理層級 | 無 Controller 管理 | 由 Controller 持續協調 |

#### 為什麼 Pod 名稱這麼長？

建立 Deployment 後，使用 `kubectl get all` 會看到：

```
pod/my-nginx-6b7f675859-abc12
```

名稱組成規則：
```
{deployment 名稱}-{ReplicaSet hash}-{Pod 隨機 ID}
```

這個設計確保：
- 同一 namespace 內 Pod 名稱唯一
- 可追溯到所屬的 Deployment 與 ReplicaSet

---

### Deployment → ReplicaSet → Pod 的層級關係

當你建立一個 Deployment 時，背後發生了什麼？

```
kubectl create deployment
         ↓
    API Server（接收請求，寫入 etcd）
         ↓
    Controller Manager（監控到新 Deployment）
         ↓
    Deployment Controller（建立 ReplicaSet）
         ↓
    ReplicaSet Controller（建立 Pod）
         ↓
    Scheduler（分配 Pod 到 node）
         ↓
    Kubelet（通知 container runtime 建立容器）
```

#### 為什麼需要 ReplicaSet 這層抽象？

這是一個關鍵的設計決策，**目的是支援滾動更新（Rolling Update）**。

當你更新 Deployment（例如更換 image 版本）：

1. **新的 ReplicaSet 被建立** — 攜帶新的 Pod spec
2. **舊的 ReplicaSet 暫時保留** — 舊版 Pod 仍在運行
3. **逐步替換** — 新 Pod 啟動、舊 Pod 終止
4. **最終結果** — 只剩下新的 ReplicaSet

> 每次變更 Pod spec（image、env、config），都會產生新的 ReplicaSet 版本。

---

### Controller Manager 的角色

Controller Manager 是 Control Plane 的核心元件，內建多個 Controller：

| Controller | 負責資源 |
| --- | --- |
| Deployment Controller | Deployment |
| ReplicaSet Controller | ReplicaSet |
| Node Controller | Node 狀態 |
| ... | ... |

每個 Controller 的工作邏輯相同：

1. 監控 etcd 中的資源定義
2. 比對「期望狀態」與「實際狀態」
3. 採取行動消除差異

> 這就是 Kubernetes 的**聲明式 API** 核心：你告訴系統「我要什麼」，系統負責「如何達成」。

---

### 清理資源

在進入下一單元前，清理建立的資源：

```bash
kubectl delete pod my-nginx
kubectl delete deployment my-nginx
```

> 可以建立同名 Pod 與 Deployment，因為它們是**不同的資源類型**。同一 namespace 中，相同資源類型不能重名。

---

## 💡 重點摘要

- **Deployment 是 Kubernetes 最常用的資源類型，類似 Swarm 的 service。**
- **Deployment 不直接建立 Pod，而是建立 ReplicaSet，再由 ReplicaSet 建立 Pod。**
- **ReplicaSet 作為 Pod spec 的版本快照，是實現滾動更新的關鍵設計。**
- **Pod 名稱格式為 `{deployment}-{replicaset-hash}-{pod-id}`，確保唯一性與可追溯性。**
- **Controller Manager 內建多個 Controller，持續協調期望狀態與實際狀態。**

---

## 🔑 關鍵字

Deployment, ReplicaSet, Controller Manager, Rolling Update, Pod Spec, etcd, Declarative API
