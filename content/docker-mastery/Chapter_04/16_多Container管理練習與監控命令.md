# 多 Container 管理練習與監控命令

## 📝 課程概述

本單元將透過實際練習熟悉多 Container 的管理操作，並學習如何監控 Container 的運行狀態。我們會使用 `top`、`inspect`、`stats` 等命令來深入了解 Container 內部的程序與資源使用情況。

## 核心觀念與實作解析

### 練習：啟動三個 Container

這個練習要求同時啟動 Nginx、Apache（httpd）、MySQL 三個 Container，並正確配置各自的 Port 與環境變數。

**MySQL Container**：

```bash
docker container run -d -p 3306:3306 --name db -e MYSQL_RANDOM_ROOT_PASSWORD=true mysql
```

- `-e`：設定環境變數
- `MYSQL_RANDOM_ROOT_PASSWORD=true`：讓 MySQL 在首次啟動時產生隨機 root 密碼

**取得隨機密碼**：

```bash
docker container logs db
```

日誌中會顯示 `GENERATED ROOT PASSWORD`，這就是隨機產生的密碼。

**Apache Container**：

```bash
docker container run -d -p 8080:80 --name webserver httpd
```

**Nginx Container**：

```bash
docker container run -d -p 80:80 --name proxy nginx
```

**驗證 Port 是否正確開啟**：

```bash
curl localhost       # Nginx
curl localhost:8080  # Apache
```

---

### 查看 Container 程序：`top`

```bash
docker container top <container_name>
```

這個命令會列出 Container 內運行的程序，類似 Linux 的 `ps` 命令。

---

### 查看 Container 配置：`inspect`

```bash
docker container inspect <container_name>
```

這會以 JSON 格式輸出 Container 的完整配置資訊，包括：
- 啟動時的參數
- 環境變數
- 網路配置
- 掛載的 Volume

> 這些配置資訊會隨著課程進度逐漸理解其含義。

---

### 即時監控資源：`stats`

```bash
docker container stats
```

這會即時顯示所有 Container 的資源使用狀況：

| 欄位 | 說明 |
|------|------|
| CONTAINER ID | Container 識別碼 |
| CPU % | CPU 使用率 |
| MEM USAGE / LIMIT | 記憶體使用量 / 上限 |
| MEM % | 記憶體使用率 |
| NET I/O | 網路輸入/輸出 |
| BLOCK I/O | 磁碟讀寫 |

> 按 `Ctrl+C` 離開即時監控畫面。

**應用場景**：
- 本地開發時確保不會耗盡機器資源
- 快速檢查是否有異常的程序佔用過多資源

---

### 清理 Container

**停止多個 Container**：

```bash
docker container stop db webserver proxy
```

**移除多個 Container**：

```bash
docker container rm db webserver proxy
```

**查看已下載的 Image**：

```bash
docker image ls
```

> Container 移除後，Image 仍會保留在本地，需要另外清理。

---

## 💡 重點摘要

- **`-e` 用於設定環境變數，MySQL Image 需要指定密碼相關設定。**
- **`docker container top` 查看程序；`inspect` 查看配置；`stats` 即時監控資源。**
- **一個 Host Port 只能對應一個 Container，多個 Container 需使用不同 Port。**
- **Container 與 Image 是分開管理的，移除 Container 不會刪除 Image。**

## 🔑 關鍵字

Container, stats, inspect, top, Environment Variable
