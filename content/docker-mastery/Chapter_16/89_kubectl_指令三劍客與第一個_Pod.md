# kubectl 指令三劍客與第一個 Pod

## 📝 課程概述

本單元正式進入 Kubernetes 命令列實作。我們將學習 `kubectl run`、`kubectl create`、`kubectl apply` 三大核心指令的差異與適用場景，並建立第一個 Pod，理解 Kubernetes 如何透過抽象層管理容器。

---

## 核心觀念與實作解析

### Kubernetes 的設計哲學：多種方式達成同一目標

Kubernetes 與 Docker 工具鏈最大的不同在於：**Kubernetes 非常「不帶意見」（unopinionated）**。

- 同一件事往往有三、四種不同的做法
- 選擇取決於你的工作流程偏好（imperative vs declarative）
- 這增加了學習曲線，但也提供了極大的彈性

> 這是 Kubernetes 的核心主題：**靈活性優先，代價是需要更多時間理解各種選項。**

---

### 三大核心指令比較

| 指令 | 用途 | 類比（Swarm 經驗） |
| --- | --- | --- |
| **kubectl run** | 建立單一 Pod（1.18 後限縮功能） | 類似 `docker run` |
| **kubectl create** | 建立各種資源類型 | 類似 `docker service create` |
| **kubectl apply** | 以 YAML 定義並應用配置 | 類似 `docker stack deploy` |

#### kubectl run 的演變

在 Kubernetes 1.18 之前，`kubectl run` 可以建立多種資源類型。但現在：

- **功能已限縮**：只能建立單一 Pod
- **官方方向**：逐步棄用其他功能，讓 `run` 專注於 Pod 建立
- **適用場景**：測試、除錯、快速驗證（非生產環境）

---

### 環境驗證：kubectl version

在開始操作前，先確認 CLI 與 API 的連線狀態：

```bash
kubectl version

# 或使用簡潔格式
kubectl version --short
```

這會顯示：
- **Client Version**：本地 kubectl 版本
- **Server Version**：Kubernetes 叢集版本

> 確保 client 與 server 版本差距在幾個 minor version 以內即可正常運作。

---

### 建立第一個 Pod

```bash
kubectl run my-nginx --image nginx
```

重要觀念：

1. **Pod 名稱是強制的** — 不像 Docker 會自動生成隨機名稱
2. **回應僅表示 API 已接收請求** — 不保證容器已啟動
3. **請求被寫入 etcd** — Controller 會持續協調直到狀態符合期望

#### 查看運行中的 Pod

```bash
kubectl get pods
```

輸出欄位說明：

| 欄位 | 意義 |
| --- | --- |
| NAME | Pod 名稱 |
| READY | 就緒容器數 / 總容器數 |
| STATUS | 運行狀態（Running 為理想狀態） |
| RESTARTS | 重啟次數（通常應為 0） |
| AGE | 運行時間 |

#### 查看所有常用資源

```bash
kubectl get all
```

> 注意：`get all` 並非真的列出「所有」資源，而是列出**開發者常用的資源類型**。Kubernetes 實際上有 70-80 種資源類型。

---

### 為什麼需要 Pod？

Pod 是 Kubernetes 獨有的概念，Docker 與 Swarm 都沒有這層抽象。

**Pod 的本質是一層抽象**：

- 一個 Pod 可包含一個或多個容器
- 同一 Pod 內的容器**共享 IP 位址**與 localhost
- 同一 Pod 內的容器**一定部署在同一個 node**

> Kubernetes **只建立 Pod，不直接建立容器**。是 Kubelet 告訴 container runtime（Docker、containerd、cri-o）建立實際的容器。

#### Pod 建立流程

```
kubectl run → API Server → etcd（儲存 Pod 定義）
                              ↓
                         Scheduler（分配 node）
                              ↓
                         Kubelet（監控到新 Pod）
                              ↓
                      Container Runtime（建立容器）
```

---

## 💡 重點摘要

- **Kubernetes 設計哲學是「多種方式達成同一目標」，學習曲線較陡但彈性極大。**
- **kubectl run 在 1.18 後只建立單一 Pod，適合測試場景而非生產環境。**
- **Pod 是 Kubernetes 最小部署單位，本質是共享 IP 與 localhost 的容器抽象層。**
- **Kubernetes 不直接建立容器，而是透過 Kubelet 告知 container runtime 建立容器。**
- **kubectl get all 列出的是開發者常用資源，並非真正意義的「所有資源」。**

---

## 🔑 關鍵字

kubectl, Pod, run, create, apply, Kubelet, Container Runtime, etcd, Scheduler
