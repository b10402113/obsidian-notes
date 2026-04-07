# 建立三節點 Swarm Cluster

## 📝 課程概述

本單元將帶領我們從單一 Node 的 Swarm 擴展到真正的多節點 Cluster。我們會探討三種建立測試環境的方式（Play with Docker、Docker Machine、雲端 VM），並實際建立一個具有三個 Manager Node 的 Swarm Cluster，體驗真正的分散式容器編排。

---

## 核心觀念與實作解析

### 為什麼需要多節點環境？

本課程第一次需要使用多台機器。單一 Node 的 Swarm 雖然可以練習基本操作，但要真正理解 Swarm 的價值，需要多個 Node：

- 體驗跨 Node 的 Service 調度
- 測試 Manager 的高可用性
- 理解分散式系統的運作

---

### 三種建立測試環境的方式

#### 方式一：Play with Docker（最簡單）

**優點：**

- 免費、無需設定
- Docker 已預裝
- 幾秒鐘就能建立多個 Node

**限制：**

- 每個 Session 只有 4 小時
- 適合一次完成整個章節的學習

**使用方式：**

1. 前往 [play-with-docker.com](https://play-with-docker.com)
2. 通過 CAPTCHA 驗證
3. 點擊「Add Instance」建立 Node（重複 3 次）

> Play with Docker 使用「Docker in Docker」技術，在 container 內運行 Docker，但使用體驗與真實 VM 幾乎相同。Node 之間可以透過友善名稱（如 `node1`、`node2`）互相溝通。

#### 方式二：Docker Machine + VirtualBox

**適合對象：** 有足夠 RAM 的本機環境

**需求：**

- VirtualBox（免費）
- 每個 VM 約需 1GB RAM
- 總共需要 3GB+ RAM

**基本指令：**

```bash
# 建立 VM
docker-machine create node1
docker-machine create node2
docker-machine create node3

# 連線到 VM
docker-machine ssh node1

# 或設定環境變數，讓本地 CLI 操作遠端 Docker
eval $(docker-machine env node1)
```

> Docker Machine 預設使用 VirtualBox，但也支援 AWS、Azure、DigitalOcean 等雲端平台。

#### 方式三：雲端 VM（DigitalOcean 範例）

老師推薦使用 DigitalOcean，原因：

- 成本低（$5-10/月/Node）
- 使用 SSD，速度快
- 體驗最接近正式環境

**建議配置：**

- OS：Ubuntu 16.04 或更新版本（最新 Docker 支援度最佳）
- 規格：$10/月（1GB RAM）較保險
- 數量：3 個 Droplet

**設定 SSH Key：**

建議設定 `~/.ssh/config`，用名稱取代 IP：

```
Host node1
    HostName 123.45.67.89
    User root
    IdentityFile ~/.ssh/id_rsa

Host node2
    HostName 123.45.67.90
    User root
    IdentityFile ~/.ssh/id_rsa
```

這樣就可以直接用 `ssh node1` 連線。

---

### 在雲端 VM 上安裝 Docker

如果使用 DigitalOcean 或自建 VM，需要先安裝 Docker：

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

> `get.docker.com` 是官方提供的安裝腳本，會自動偵測 Linux 發行版並安裝最新版本。

---

### 建立 Swarm Cluster

#### Step 1：在第一個 Node 初始化 Swarm

```bash
docker swarm init --advertise-addr <public-ip>
```

**為什麼需要 `--advertise-addr`？**

雲端 VM 通常有多個網卡（public IP、private IP）。我們需要明確指定讓其他 Node 可以連線的 IP 位址。

**輸出重要資訊：**

```
Swarm initialized: current node (xyz...) is now a manager.

To add a worker to this swarm, run the following command:
    docker swarm join --token SWMTKN-1... <ip>:2377
```

#### Step 2：加入其他 Node

**Worker Token（加入為 Worker）：**

```bash
# 在 node2 上執行
docker swarm join --token SWMTKN-1... <ip>:2377
```

**Manager Token（加入為 Manager）：**

```bash
# 在 node1 上取得 manager token
docker swarm join-token manager

# 在 node3 上執行輸出的指令
docker swarm join --token SWMTKN-1... <ip>:2377
```

---

### 提升 Worker 為 Manager

如果已經以 Worker 身分加入，可以事後提升：

```bash
# 在 Manager 上執行
docker node update --role manager node2
```

---

### 驗證 Cluster 狀態

```bash
docker node ls
```

輸出範例：

```
ID                           HOSTNAME   STATUS   AVAILABILITY   MANAGER STATUS
abc123...                    * node1    Ready    Active         Leader
def456...                    node2      Ready    Active         Reachable
ghi789...                    node3      Ready    Active         Reachable
```

**欄位說明：**

- `*`：你目前連線的 Node
- `Leader`：主要 Manager
- `Reachable`：可成為 Leader 的備援 Manager

---

### Manager vs. Worker 的權限差異

**Worker 的限制：**

- 無法執行 `docker node ls` 等 Swarm 管理指令
- 只能執行被分配的任務

**Manager 的能力：**

- 可以管理整個 Swarm
- 可以執行所有 `docker service` 指令
- 可以檢視所有 Node 的狀態

> 通常建議設定 3 或 5 個 Manager，以確保 Raft Consensus 的高可用性。

---

### Join Token 的管理

**查看 Token：**

```bash
docker swarm join-token worker
docker swarm join-token manager
```

**Token 的特性：**

- 儲存在 Raft Database 中（加密）
- 不需要手動記錄
- 可以隨時查詢

**輪換 Token（安全措施）：**

```bash
docker swarm join-token --rotate worker
```

當有 Node 可能有安全漏洞，或 Token 意外洩露時，應該輪換 Token。

---

### 在多節點 Swarm 上部署 Service

```bash
# 建立 3-replica 的 Service
docker service create --replicas 3 alpine ping 8.8.8.8

# 查看 Service 狀態
docker service ls

# 查看所有 Task
docker service ps <service_name>

# 查看特定 Node 上的 Task
docker node ps node2
```

---

### 工作習慣：從單一 Manager 管理整個 Swarm

一旦 Swarm 建立完成，大多數操作只需要在單一 Manager Node 上執行即可。不需要頻繁切換 Node。

```bash
# 從 node1 查看 node2 的狀態
docker node ps node2

# 從 node1 查看所有 Task
docker service ps <service_name>
```

---

## 💡 重點摘要

- **Play with Docker 是最快的練習方式；雲端 VM 提供最接近正式環境的體驗。**
- **`docker swarm init --advertise-addr` 需要指定其他 Node 可連線的 IP 位址。**
- **Worker 和 Manager 有不同的權限，Worker 無法執行 Swarm 管理指令。**
- **建議設定 3 個 Manager 以確保高可用性，Manager 之間透過 Raft 保持一致性。**
- **Join Token 儲存在 Raft Database 中，可隨時查詢或輪換以確保安全。**

---

## 🔑 關鍵字

Swarm Cluster, Manager Node, Join Token, Raft Consensus, High Availability
