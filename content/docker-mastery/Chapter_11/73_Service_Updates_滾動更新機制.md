# Service Updates 滾動更新機制

## 📝 課程概述

本單元深入探討 Docker Swarm 的 Service Updates 功能。這是生產環境中最重要的操作之一，讓我們能在不中斷服務的情況下更新容器配置、image 版本或進行擴縮容。課程詳細說明滾動更新的運作機制與實務操作技巧。

---

## 核心觀念與實作解析

### 滾動更新（Rolling Update）的核心概念

Swarm 的更新功能採用 **Rolling Update Pattern**：

- **一次更新一個 Replica**：預設逐個替換容器
- **先移除舊容器，再建立新容器**：確保資源正確釋放
- **等待新容器健康後才繼續下一個**：降低服務中斷風險

> **重要觀念**：Orchestrator 只能「限制（limit）」停機時間，無法「預防（prevent）」停機。真正的零停機需要完善的測試來確保。

### 哪些更新會觸發容器替換？

| 更新類型 | 是否替換容器 |
|---------|-------------|
| 更改 image | ✅ 是 |
| 更改環境變數 | ✅ 是 |
| 更改 port | ✅ 是 |
| 更改 label/metadata | ❌ 否（只更新 metadata） |

---

### docker service update 命令選項

`docker service update` 擁有超過 77 個選項，常用選項包括：

#### 新增/移除多值選項

```bash
# 新增環境變數
docker service update --env-add NODE_ENV=production myservice

# 移除環境變數
docker service update --env-rm NODE_ENV myservice

# 新增 port
docker service update --publish-add 8080:80 myservice

# 移除 port
docker service update --publish-rm 8080 myservice
```

#### 常用獨立命令

Docker 將高頻使用的功能獨立為命令：

```bash
# 擴縮容
docker service scale web=5 api=3

# 回滾
docker service rollback myservice
```

---

### Stack Updates：沒有獨立的 Update 命令

Stack 的更新方式非常簡單：

```bash
# 第一次部署
docker stack deploy -c docker-compose.yml myapp

# 更新：編輯檔案後，再次執行相同命令
docker stack deploy -c docker-compose.yml myapp
```

> Swarm 會自動比對差異，只更新有變更的部分。

---

### 實作範例

#### 建立測試 Service

```bash
docker service create -p 8088:80 --name web nginx:1.13
```

#### 擴展 Replicas

```bash
docker service scale web=5
```

#### 滾動更新 Image

```bash
docker service update --image nginx:1.12 web
```

> Docker 不會判斷 image 版本新舊，只會比對 image 名稱是否相同。

#### 更新 Port

**注意**：無法直接修改 port，必須同時移除舊的、新增新的：

```bash
docker service update \
  --publish-rm 8088 \
  --publish-add 9090:80 \
  web
```

#### 強制更新以重新平衡

當節點負載不均時，可以強制更新讓 Scheduler 重新分配：

```bash
docker service update --force web
```

這會觸發所有 Tasks 重新建立，Scheduler 會選擇負載最低的節點。

---

### 更新失敗的考量

**Orchestrator 無法判斷的情境**：

- Database 更新對 Web App 的影響
- WebSocket 長連接的中斷處理
- 持久化連線的狀態維護

> **最佳實踐**：針對每個應用程式的特性進行充分測試，了解更新對使用者的實際影響。

---

## 💡 重點摘要

- **Swarm 採用 Rolling Update 機制，預設一次更新一個 Replica，逐步替換容器。**
- **Orchestrator 只能限制停機時間，預防停機需要透過測試來確保。**
- **Stack 更新只需再次執行 `docker stack deploy`，無需額外命令。**
- **`--force` 選項可強制重新建立 Tasks，用於重新平衡節點負載。**
- **多值選項使用 `--xxx-add` 和 `--xxx-rm` 來新增和移除。**

---

## 🔑 關鍵字

Service Update, Rolling Update, Swarm, Scale, Rollback, Replicas
