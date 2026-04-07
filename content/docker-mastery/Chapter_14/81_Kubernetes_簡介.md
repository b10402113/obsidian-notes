# Kubernetes 簡介

## 📝 課程概述

本單元介紹 Kubernetes 這個在 Swarm 之後應該學習的 Container Orchestrator。我們將了解 Kubernetes 的起源、與 Docker 的關係、核心工具 `kubectl`，以及各種 Kubernetes 發行版的選擇策略。

## 核心觀念與實作解析

### 什麼是 Kubernetes？

Kubernetes 是一個流行的 **Container Orchestrator**（容器編排器）。回顧編排器的定義：

> 編排器負責將你要求運行的 Container，分配到一組伺服器（節點）上執行。

Kubernetes 於 2015 年由 Google 釋出，目前由全球開源社群維護，Google 仍是主要貢獻者之一。

### Kubernetes 與 Docker 的關係

Kubernetes 本質上是一組 API，透過運行在 Container 中的應用程式來：

1. 管理一組伺服器叢集
2. 在 Docker（或其他 Container Runtime）上執行你的 Container

```
┌─────────────────────────────────────┐
│           Kubernetes API            │
│    (一組運行在 Container 中的服務)    │
├─────────────────────────────────────┤
│         Container Runtime           │
│           (預設為 Docker)            │
├─────────────────────────────────────┤
│              Node/OS                │
└─────────────────────────────────────┘
```

> Kubernetes **不取代** Container Runtime，而是在其上增加管理多節點系統的能力。

### 核心工具：kubectl

在 Kubernetes 中，主要使用的 CLI 工具是 `kubectl`（讀作 kube-control）：

| 工具 | 用途 |
|------|------|
| `docker` | Docker 命令 |
| `kubectl` | Kubernetes 命令 |

> 過去有各種唸法（kube-cuddle、kube-ctl），但官方現已統一為 **kube-control**。

### 如何取得 Kubernetes？

取得 Kubernetes 的方式可分為兩大類：

#### 1. 雲端託管服務 (Kubernetes as a Service)

各大雲端供應商提供託管 Kubernetes：

- AWS EKS
- Google GKE
- Azure AKS
- 其他雲端平台

你會獲得：
- Kubernetes API 端點
- 可使用本地工具或雲端 GUI 部署與管理

#### 2. 發行版 (Distributions)

類似 Linux 發行版的概念：

- 核心都是相同的 **Upstream Kubernetes**（開源社群版本）
- 各廠商在此之上打包自己的工具與功能

**常見發行版**：

| 發行版 | 供應商 |
|--------|--------|
| Docker Enterprise | Docker |
| OpenShift | Red Hat |
| Rancher | SUSE |
| PKS | VMware |
| Canonical Kubernetes | Ubuntu |

### 選擇發行版的考量

#### 憑證認證的重要性

應選擇 **CNCF Certified Kubernetes** 發行版：

- 保證 API 完全相容
- YAML 檔案可在不同發行版間移植
- 工作負載可順暢遷移

#### 發行版提供的額外價值

Kubernetes 開源版本需要許多額外元件才能在生產環境使用：

- 自訂認證方案
- Web 管理介面
- 網路方案
- 儲存方案
- 監控與日誌

發行版通常會預先整合這些工具，開箱即用。

### Upstream vs. 發行版

```
Upstream Kubernetes (GitHub)
        │
        ├── Docker Enterprise
        ├── OpenShift
        ├── Rancher
        └── ...
```

**建議**：

- **學習階段**：使用 Upstream 版本或輕量工具
- **生產環境**：使用發行版，獲得支援與整合工具

> 不要直接 Clone Kubernetes GitHub Repository 在生產環境使用，這不是正確的方式。

### 版本追蹤

發行版通常會追蹤 Upstream 版本：

- 過去可能落後數個版本
- 現在競爭激烈，通常只落後一個版本
- 選擇時可注意支援的 Kubernetes 版本

## 💡 重點摘要

- Kubernetes 是 Container Orchestrator，運行在 Docker 等 Runtime 之上
- `kubectl` 是主要的管理工具，官方讀音為 kube-control
- 雲端託管服務與發行版是兩種主要的取得方式
- 選擇 CNCF 認證發行版可確保 API 相容性
- 學習用 Upstream，生產用發行版

## 🔑 關鍵字

Kubernetes, kubectl, Orchestrator, Distribution, Upstream
