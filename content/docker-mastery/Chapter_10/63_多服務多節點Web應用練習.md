# 多服務多節點 Web 應用練習

## 📝 課程概述

本單元是一個整合練習，要求學員使用 Docker 的 Example Voting App 來實作多層架構應用。透過這個練習，你將學會如何設計並部署一個包含五個服務的分散式系統，理解真實世界中多團隊協作的容器化架構。

## 核心觀念與實作解析

### Example Voting App 架構

這是一個經典的分散式投票應用，包含五個不同技術棧的服務：

```
┌─────────────────────────────────────────────────────────────┐
│                        Frontend Network                     │
│                                                             │
│   ┌─────────┐                    ┌─────────────────────┐   │
│   │  Vote   │ (Python)           │       Redis         │   │
│   │  :80    │◄───────────────────│    (Key-Value)      │   │
│   └─────────┘                    └──────────┬──────────┘   │
│        │                                    │              │
└────────┼────────────────────────────────────┼──────────────┘
         │                                    │
         │                            ┌───────▼────────┐
         │                            │     Worker     │
         │                            │    (.NET)      │
         │                            └───────┬────────┘
         │                                    │
┌────────┼────────────────────────────────────┼──────────────┐
│        │                            Backend Network        │
│        │                                    │              │
│   ┌────▼────────┐                    ┌─────▼──────────┐   │
│   │   Result    │ (Node.js)          │    Postgres    │   │
│   │    :5001    │◄───────────────────│     (DB)       │   │
│   └─────────────┘                    └────────────────┘   │
│                                          Volume: db-data   │
└─────────────────────────────────────────────────────────────┘
```

---

### 服務說明

| 服務 | 技術棧 | 網路 | 功能 |
|------|--------|------|------|
| **vote** | Python | frontend | 投票前端（貓 vs 狗） |
| **redis** | Redis | frontend | 暫存投票佇列 |
| **worker** | .NET | frontend + backend | 處理投票並寫入 DB |
| **db** | PostgreSQL | backend | 儲存投票結果 |
| **result** | Node.js | backend | 即時顯示投票結果（WebSocket） |

---

### 部署要求

**網路設計**：

```bash
# 建立兩個 Overlay Network
docker network create --driver overlay frontend
docker network create --driver overlay backend
```

> 為什麼要兩個網路？前端服務不應直接存取資料庫，這是安全性隔離。

**Volume 設計**：

資料庫需要持久化儲存，但 `-v` 參數在 `docker service create` 中不支援，需使用 `--mount`：

```bash
--mount type=volume,source=db-data,target=/var/lib/postgresql/data
```

---

### 服務名稱的重要性

這個應用有個關鍵特性：**服務名稱是硬編碼的**：

- `vote` 服務會查找名為 `redis` 的 DNS 記錄
- `worker` 會查找 `redis` 和 `db`
- `result` 會查找 `db`

> 這意味著你必須使用指定的服務名稱，否則 DNS 解析會失敗。

---

### 部署步驟摘要

**1. 建立網路**：

```bash
docker network create --driver overlay frontend
docker network create --driver overlay backend
```

**2. 建立 Vote 服務**：

```bash
docker service create --name vote --network frontend --replicas 2 -p 80:80 dockersamples/examplevotingapp_vote
```

**3. 建立 Redis 服務**：

```bash
docker service create --name redis --network frontend --replicas 2 redis:3.2
```

**4. 建立 Worker 服務**（連接兩個網路）：

```bash
docker service create --name worker --network frontend --network backend dockersamples/examplevotingapp_worker
```

**5. 建立資料庫服務**：

```bash
docker service create --name db --network backend --mount type=volume,source=db-data,target=/var/lib/postgresql/data postgres:9.4
```

**6. 建立 Result 服務**：

```bash
docker service create --name result --network backend -p 5001:80 dockersamples/examplevotingapp_result
```

---

### 驗證部署

```bash
# 查看所有服務
docker service ls

# 查看特定服務的 Task
docker service ps <service_name>
```

**測試應用**：
- `http://<node-ip>:80` — 投票頁面
- `http://<node-ip>:5001` — 結果頁面

---

## 💡 重點摘要

- **多層架構需要多個 Overlay Network 來實現安全隔離。**
- **服務名稱決定 DNS 解析，若應用硬編碼名稱則必須使用相同名稱。**
- **Worker 需要連接兩個網路，使用兩次 `--network` 參數。**
- **使用 `--mount` 而非 `-v` 來設定 Service 的 Volume。**

## 🔑 關鍵字

Service, Overlay, Volume, Network, DNS
