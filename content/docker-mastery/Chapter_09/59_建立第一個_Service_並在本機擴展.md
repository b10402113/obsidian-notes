# 建立第一個 Service 並在本機擴展

## 📝 課程概述

本單元是 Swarm Mode 的實作入門。我們將從單一 Node 的 Swarm 開始，學習如何建立 Service、查詢 Service 狀態、動態調整 Replica 數量，以及理解 Swarm 如何自動恢復失敗的 Container。這些操作模式與傳統 `docker run` 有本質上的不同。

---

## 核心觀念與實作解析

### 啟用 Swarm Mode

首先確認 Swarm 是否已啟用：

```bash
docker info
```

在輸出中找到 `Swarm: inactive`，表示 Swarm 尚未啟用。

**啟用 Swarm：**

```bash
docker swarm init
```

這個指令只需要約 0.5 秒就完成了，但背景發生了許多事情。

---

### `docker swarm init` 背景做了什麼？

老師詳細解釋了這個簡單指令背後的複雜操作：

**安全基礎建設：**

1. **建立 Root Certificate**：用於建立 Swarm 的信任基礎
2. **簽發 Manager 憑證**：為第一個 Manager Node 建立專屬憑證
3. **產生 Join Tokens**：讓其他 Node 可以加入 Swarm 的金鑰

**資料庫與配置：**

4. **啟用 Swarm API**
5. **建立 Raft Consensus Database**：
   - 儲存 Swarm 配置
   - 加密儲存於磁碟
   - 確保跨 Node 的一致性

**關鍵設計決策：**

Swarm 不需要額外的 Key-Value Store（如 etcd、Consul）。傳統的編排系統需要額外建置配置資料庫，而 Swarm 將這個功能直接內建於 Docker Engine 中。

> Swarm 的所有流量（跨 Node 通訊）都會被加密，確保安全性。

---

### 查看 Swarm 狀態

**列出所有 Node：**

```bash
docker node ls
```

輸出範例：

```
ID                           HOSTNAME   STATUS   AVAILABILITY   MANAGER STATUS
xyz123...                    * node1    Ready    Active         Leader
```

- `*` 表示你目前連線的 Node
- `Leader` 表示這是主要 Manager（一個 Swarm 同時間只有一個 Leader）

**Node 指令用途：**

- 加入/移除 Node
- 提升/降級 Node 角色（Worker ↔ Manager）

---

### 建立 Service：取代 docker run

在 Swarm 中，`docker service create` 取代了 `docker run`：

```bash
docker service create alpine ping 8.8.8.8
```

**這個指令的意思：**

- 使用 `alpine` image
- 執行 `ping 8.8.8.8`（只是讓 container 有事情做，方便觀察）

**輸出差異：**

- `docker run` 回傳 Container ID
- `docker service create` 回傳 **Service ID**

---

### 查看 Service 狀態

**列出所有 Service：**

```bash
docker service ls
```

輸出範例：

```
ID                  NAME            MODE        REPLICAS
abc123              frosty_mountain replicated  1/1
```

**REPLICAS 欄位的意義：**

- `1/1` = 左邊是「實際運行數量」/ 右邊是「期望數量」
- Orchestrator 的目標就是讓這兩個數字相等

**查看 Service 的 Task（Container）：**

```bash
docker service ps <service_name_or_id>
```

這會顯示該 Service 下的所有 Task，類似 `docker container ls`，但多了 `NODE` 欄位顯示運行在哪個 Node。

---

### 動態擴展 Service

**增加 Replica 數量：**

```bash
docker service update <service_id> --replicas 3
```

**驗證：**

```bash
docker service ls
# 輸出：3/3

docker service ps <service_name>
# 顯示 3 個 Task
```

> 如果動作夠快，你可能會看到 `0/3` → `1/3` → `2/3` → `3/3` 的過程。

---

### `docker service update` vs `docker update`

這是兩個完全不同的指令：

| 指令 | 用途 |
|------|------|
| `docker update` | 更新單一 Container 的資源限制（CPU、RAM） |
| `docker service update` | 更新 Service 的配置，支援 Rolling Update |

**Service Update 的優勢：**

如果我們有一個 3-replica 的 Service，當我們進行更新時：

1. Swarm 會逐一更新每個 Container
2. 確保隨時都有 Container 在運行
3. 實現 Blue-Green Deploy（零停機更新）

---

### Swarm 的自我修復能力

這是 Swarm 最強大的特性之一。讓我們測試一下：

**情境：手動刪除 Container**

```bash
# 找到一個 container ID
docker container ls

# 強制刪除它
docker container rm -f <container_id>
```

**觀察結果：**

```bash
docker service ls
# 可能短暫顯示 2/3

# 幾秒後再查看
docker service ls
# 恢復為 3/3
```

**Service PS 顯示歷史：**

```bash
docker service ps <service_name>
```

輸出會顯示：

- 舊的 Task 標記為 `Failed` 或 `Shutdown`
- 新的 Task 已經啟動

> 這就是 **Orchestration** 的核心價值：當實際狀態偏離期望狀態時，系統會自動修正。

---

### Service vs. Container 操作思維

**傳統 `docker run` 思維：**

- 直接對 Container 下指令
- Container 刪除就是刪除了，不會自動恢復

**Swarm `docker service` 思維：**

- 我們向 Orchestrator 提交「期望狀態」
- Orchestrator 持續監控並確保狀態符合期望
- 支援 Rollback、失敗恢復、智慧調度

---

### 刪除 Service

```bash
docker service rm <service_name>
```

**有趣的觀察：**

執行後立即查看：

```bash
docker service ls
# 空的

docker container ls
# 可能還看到 containers 正在被清理
```

這顯示了 Orchestrator 在背景非同步執行清理工作。

---

## 💡 重點摘要

- **`docker swarm init` 一行指令完成 PKI 憑證、Raft Database、加密通訊等安全基礎建設。**
- **Swarm 不需要額外的 Key-Value Store，配置資料庫已內建於 Docker Engine。**
- **`docker service create` 取代 `docker run`，讓我們定義 Service 而非個別 Container。**
- **REPLICAS 顯示「實際/期望」數量，Orchestrator 會持續確保兩者相等。**
- **Swarm 具備自我修復能力，當 Container 失敗或被刪除時會自動重建。**

---

## 🔑 關鍵字

Swarm Init, Service Create, Service Update, Replicas, Self-Healing
