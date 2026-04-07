# ENTRYPOINT 與 CMD 組合使用

## 📝 課程概述

本單元展示 `ENTRYPOINT` 與 `CMD` 組合使用的強大威力。當兩者同時設定時，Docker 會將它們合併成一個指令執行。這開啟了兩個重要應用場景：建立 CLI 工具容器，以及在主程序啟動前執行 Startup Script。

---

## 核心觀念與實作解析

### ENTRYPOINT + CMD 的合併機制

當 Dockerfile 同時設定兩者時：

```dockerfile
ENTRYPOINT ["curl"]
CMD ["--help"]
```

Container 啟動時執行的指令是：

```bash
curl --help
```

> Docker 將 `ENTRYPOINT` 與 `CMD` 用空格連接，形成完整的執行指令。

---

### 應用場景一：CLI 工具容器

#### 範例：curl 容器

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

ENTRYPOINT ["curl"]
CMD ["--help"]
```

**使用方式：**

```bash
# 顯示 curl help
docker run mycurl

# 實際執行 curl https://example.com
docker run mycurl https://example.com

# 加上選項
docker run mycurl -I https://example.com
```

**設計邏輯：**
- `ENTRYPOINT` 固定為 `curl`
- `CMD` 提供預設值 `--help`
- 使用者可在 `docker run` 後加上 URL 或選項，覆蓋 `CMD`

#### 為什麼這樣設計？

如果只用 `CMD`：
```bash
docker run mycurl https://example.com
# 需要覆蓋整個 CMD，包括 curl 本身
```

使用 `ENTRYPOINT + CMD`：
```bash
docker run mycurl https://example.com
# 只需覆蓋 CMD 部分，ENTRYPOINT 固定
```

> **這讓 Container 像一個原生 CLI 工具。**

---

### 應用場景二：Startup Script

#### 為什麼需要 Startup Script？

許多應用程式在啟動前需要：
- 初始化資料庫
- 設定設定檔
- 建立目錄
- 檢查環境變數

#### 錯誤做法：在 CMD 中執行 Script

```dockerfile
# ❌ 錯誤做法
CMD ["sh", "-c", "init.sh && python app.py"]
```

**問題：**
- `python app.py` 是 Shell 的子程序
- Signal（SIGTERM）會發給 Shell，不會傳遞給 Python
- Container 無法優雅關閉

#### 正確做法：ENTRYPOINT Script + CMD

```dockerfile
COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["python", "app.py"]
```

**docker-entrypoint.sh：**

```bash
#!/bin/bash
set -e

# 執行初始化工作
init_database
setup_config

# 關鍵：將執行權交給 CMD
exec "$@"
```

**`exec "$@"` 的作用：**
- `$@` 取得 CMD 的內容（`python app.py`）
- `exec` 將當前 Shell 替換為該程序
- Python 成為 PID 1，可正確接收 Signal

---

### MySQL 官方 Image 的實例

MySQL 官方 Image 就是這個模式：

```dockerfile
COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["mysqld"]
```

**docker-entrypoint.sh 做的事：**
- 初始化資料庫檔案
- 設定 root 密碼
- 建立使用者
- 最後 `exec "$@"` 啟動 mysqld

**額外好處：覆蓋 CMD 取得 Shell**

```bash
docker run -it mysql sh
# ENTRYPOINT script 仍會執行
# 但 CMD 被覆蓋為 sh，mysqld 不會啟動
# 可使用 mysql 工具（mysqldump、mysqlimport）
```

---

### 驗證：覆蓋 CMD 後的程序狀態

```bash
# 啟動 MySQL 但覆蓋 CMD
docker run --name myshell -it mysql sh

# 在另一個終端機查看程序
docker top myshell
# 只看到 sh，沒有 mysqld
```

---

## 💡 重點摘要

- **`ENTRYPOINT` 與 `CMD` 組合時，Docker 將兩者用空格連接成完整指令。**
- **CLI 工具容器：`ENTRYPOINT` 放程式名稱，`CMD` 放預設參數，使用者可輕鬆覆蓋參數。**
- **Startup Script：ENTRYPOINT Script 執行初始化後，用 `exec "$@"` 將執行權交給 CMD。**
- **直接在 CMD 執行 Script 會讓主程序成為子程序，無法正確接收 Signal。**
- **`exec "$@"` 是 Shell 技巧，將 Script 替換為 CMD 指定的程序，確保該程序成為 PID 1。**

---

## 🔑 關鍵字

ENTRYPOINT, CMD, Startup Script, exec, PID 1, Signal, CLI Tool
