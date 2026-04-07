# Swarm Mode 內建容器編排系統

## 📝 課程概述

本單元正式進入容器編排（Orchestration）的世界。當我們需要跨多台 Server 部署、擴展和維護數十甚至數千個 Container 時，單純的 `docker run` 已不足以應付。Docker Swarm Mode 是 Docker 在 2016 年內建的編排解決方案，讓我們能夠用現有的 Docker 知識，輕鬆管理跨多節點的容器生命週期。

---

## 核心觀念與實作解析

### 容器規模化帶來的新挑戰

容器的承諾是：我們可以像使用 PaaS（如 Heroku）一樣，在任何硬體上部署應用程式——無論是本地機器、AWS、Azure、DigitalOcean 或其他雲端平台。

但當容器數量從幾個成長到數十、數百甚至數千個，並且需要跨多台 Server 運行時，就會遇到一系列新問題：

**核心問題清單：**

- 如何 Scale out/up？
- 如何確保 Container 失敗後自動重啟？
- 如何在不中斷服務的情況下更新 Container（Blue-Green Deploy）？
- 如何知道 Container 運行在哪個 Node 上？
- 如何處理跨 Node 的網路通訊？
- 如何確保 Container 只在授權的機器上運行？
- 如何安全地儲存機密資訊（Secrets、Passwords）？

> 這些問題對於 Netflix 這種擁有大量工程師的大型組織來說或許不是問題，但對於小型團隊或獨立開發者來說，卻是巨大的挑戰。

---

### Docker Swarm Mode 的演進

**Swarm Classic vs. Swarm Mode：**

老師特別澄清了一個常見誤解：

| 版本 | 時間 | 特性 |
|------|------|------|
| Swarm Classic | Docker 1.12 之前 | 獨立的 add-on container，只是重複執行 `docker run` 到多台 Server |
| Swarm Mode | Docker 1.12+（2016） | 內建於 Docker Engine，完整的編排解決方案 |

**重要里程碑：**

- **2016 DockerCon**：發布 Swarm Kit，一套完整的工具庫
- **2017 1.13 版**：新增 Stacks 和 Secrets 功能

> Swarm Mode 預設是**關閉的**，這是設計決策——確保不影響現有的 Docker 環境和其他編排工具。

---

### Swarm 架構核心概念

#### Manager Nodes 與 Worker Nodes

Swarm 採用 Manager-Worker 架構：

**Manager Nodes（藍色）：**

- 擁有本地 **Raft Database**，儲存整個 Swarm 的配置
- 負責調度、編排和管理工作
- 可以同時是 Worker（Manager = Worker + 管理權限）
- 通常建議 3 或 5 個 Manager 以確保高可用性

**Worker Nodes（綠色）：**

- 接收 Manager 的指令並執行任務
- 向 Manager 回報狀態
- 可以被提升為 Manager

> 每個 Node 可以是實體機器或 VM，運行 Linux 或 Windows Server，只要安裝了 Docker 即可。

#### Raft Consensus Database

這是 Swarm 的核心元件：

- 分散式共識資料庫，確保配置一致性
- 在所有 Manager 之間複製
- 加密傳輸，確保完整性與信任

---

### Service vs. Container 的概念轉變

這是理解 Swarm 最重要的觀念轉換：

**`docker run` 的限制：**

- 只能在單一 host 上建立單一 container
- 沒有 Scale 的概念
- 無法自動恢復失敗的 container

**`docker service` 的解決方案：**

Swarm 引入了新的抽象層：

```
Service → Task(s) → Container(s)
```

| 概念 | 說明 |
|------|------|
| **Service** | 我們定義的「期望狀態」，例如「3 個 Nginx replica」 |
| **Task** | Swarm 的調度單位，每個 Task 對應一個 Container |
| **Container** | 實際運行的程序 |

**範例：**

```bash
docker service create --replicas 3 nginx
```

這個指令告訴 Swarm：「我需要 3 個 Nginx container」。Manager 會自動決定將它們分配到哪些 Node 上（預設會盡量分散）。

---

### Swarm 內建的背景服務

Swarm 不只是簡單地執行指令，它有一整套背景服務在運作：

| 服務 | 功能 |
|------|------|
| **Scheduler** | 決定 Task 應該運行在哪個 Node |
| **Dispatcher** | 分配工作給 Worker |
| **Allocator** | 分配資源（網路、volume 等） |
| **Orchestrator** | 協調所有操作，確保實際狀態符合期望狀態 |

**工作流程：**

1. Worker 持續向 Manager 回報狀態
2. Manager 比對「期望狀態」與「實際狀態」
3. 如有差異，進行 Reconciliation（協調修復）

---

### Swarm 的設計哲學

**Pets vs. Cattle 類比：**

- **Pets（寵物）**：傳統伺服器思維，每台機器都有名字，需要細心照料
- **Cattle（牲畜）**：Swarm 思維，Node 只是編號，不需要個別關注

在 Swarm 中，我們不直接操作個別 Node，而是向 Swarm 提交「需求」，讓編排系統自動處理細節。

> 我們不必知道 Container 運行在哪個 Node 上，只需要信任 Swarm 會確保它們在運行。

---

## 💡 重點摘要

- **Swarm Mode 是 Docker 1.12 內建的容器編排解決方案，讓小型團隊也能輕鬆管理大規模容器部署。**
- **Swarm 採用 Manager-Worker 架構，Manager 透過 Raft Database 維護配置一致性。**
- **`docker service` 取代 `docker run`，讓我們定義「期望狀態」而非逐一建立 container。**
- **Service → Task → Container 的抽象層，讓 Swarm 能夠自動調度、擴展和恢復容器。**
- **Swarm 內建 Scheduler、Orchestrator 等背景服務，持續確保實際狀態符合期望狀態。**

---

## 🔑 關鍵字

Swarm Mode, Service, Task, Manager Node, Raft Database
