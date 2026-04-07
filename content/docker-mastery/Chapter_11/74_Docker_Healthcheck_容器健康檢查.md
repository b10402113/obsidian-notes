# Docker Healthcheck 容器健康檢查

## 📝 課程概述

本單元介紹 Docker Healthcheck 功能，這是 Docker 1.12 版本引入的重要特性。Healthcheck 讓 Docker 能夠判斷容器內的應用程式是否真正「健康運作」，而不僅僅是確認程序是否還在執行。這對於生產環境的服務可靠性至關重要。

---

## 核心觀念與實作解析

### 為什麼需要 Healthcheck？

在 Healthcheck 出現之前，Docker 只能判斷：

- **程序是否還在執行**（Process is running）
- **無法判斷應用程式是否正常運作**

> 例如：Web Server 程序還在執行，但持續返回 500 錯誤，Docker 仍然認為容器是「健康」的。

### Healthcheck 的運作機制

Healthcheck 會在**容器內部**執行命令，並根據 **Exit Code** 判斷健康狀態：

| Exit Code | 狀態 |
|-----------|------|
| 0 | Healthy（健康） |
| 1 | Unhealthy（不健康） |

**重要**：Docker 只認 Exit Code 1 為不健康，其他非 0 的 Exit Code 不會觸發不健康狀態。

### 三種健康狀態

容器會經歷三個健康狀態：

1. **starting**：啟動後的前 30 秒（預設），尚未執行第一次 Healthcheck
2. **healthy**：Healthcheck 返回 Exit Code 0
3. **unhealthy**：Healthcheck 返回 Exit Code 1

---

### 預設參數與可調整選項

```dockerfile
HEALTHCHECK --interval=30s --timeout=30s --start-period=30s --retries=3 \
  CMD curl -f http://localhost/ || exit 1
```

| 參數 | 預設值 | 說明 |
|------|--------|------|
| `--interval` | 30s | 每隔多久執行一次 Healthcheck |
| `--timeout` | 30s | 命令執行逾時時間 |
| `--start-period` | 30s | 啟動緩衝期，期間不會標記為 unhealthy |
| `--retries` | 3 | 連續失敗幾次才標記為 unhealthy |

> **start-period 的價值**：對於 Java App、Database 等啟動較慢的應用，可以設定較長的緩衝期（如 2 分鐘），避免誤判為不健康。

---

### 各種配置方式

#### 1. Dockerfile 中定義

```dockerfile
# 基本語法
HEALTHCHECK CMD curl -f http://localhost/ || exit 1

# 帶選項的語法
HEALTHCHECK --interval=1m --timeout=10s --retries=3 \
  CMD curl -f http://localhost/ping || exit 1
```

#### 2. docker run 時指定

```bash
docker run --name p2 \
  --health-cmd="pg_isready -U postgres" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  -d postgres
```

#### 3. Compose / Stack 檔案中定義

```yaml
version: "3.4"
services:
  web:
    image: nginx
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 2m  # 需要 version 3.4 以上
```

---

### 實務範例

#### Nginx 靜態網站

```dockerfile
HEALTHCHECK --interval=1m --timeout=10s \
  CMD curl -f http://localhost/ || exit 1
```

#### PostgreSQL

```bash
# pg_isready 是 PostgreSQL 內建工具
docker run --health-cmd="pg_isready -U postgres" postgres
```

#### PHP-FPM + Nginx

```dockerfile
HEALTHCHECK --interval=30s \
  CMD curl -f http://localhost/ping || exit 1
```

> `/ping` 是 PHP-FPM 的內建 status URL，需要在 php-fpm.conf 中啟用。

#### Elasticsearch

```bash
docker run --health-cmd="curl -f http://localhost:9200/_cluster/health || exit 1" \
  elasticsearch
```

---

### Healthcheck 與 Swarm 的整合

#### docker run 的行為

`docker run` **不會自動採取行動**：
- 只會在 `docker container ls` 顯示狀態
- 可以透過 `docker container inspect` 查看歷史記錄

#### Swarm Service 的行為

Swarm 會**主動採取行動**：
- 當容器被標記為 unhealthy，會自動替換為新的 Task
- 在 Rolling Update 過程中，會等待新容器通過 Healthcheck 才繼續下一個

> 這就是為什麼在生產環境應該善用 Healthcheck —— 它讓 Swarm 能夠真正判斷服務是否可用。

---

### 查看 Healthcheck 狀態

```bash
# 查看容器狀態（包含健康狀態）
docker container ls

# 查看詳細的健康檢查歷史
docker container inspect <container_name>
```

`docker container inspect` 會顯示：
- 目前健康狀態
- 最近 5 次 Healthcheck 的執行結果
- 上次執行時間與輸出內容

---

## 💡 重點摘要

- **Healthcheck 讓 Docker 能判斷應用程式是否真正健康，而不僅是程序是否還活著。**
- **容器有三種健康狀態：starting → healthy / unhealthy。**
- **Docker 只認 Exit Code 0 為健康，Exit Code 1 為不健康。**
- **`start-period` 對啟動較慢的應用（如 Java、Database）非常重要。**
- **Swarm 會根據 Healthcheck 結果自動替換不健康的容器。**

---

## 🔑 關鍵字

Healthcheck, Swarm, Exit Code, start-period, pg_isready, Rolling Update
