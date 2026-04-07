# Docker Compose CLI 實戰操作

## 📝 課程概述

本單元深入探討 Docker Compose CLI 工具的實際操作。我們將學習如何使用 `docker-compose up` 和 `docker-compose down` 兩個核心指令，以及 Compose 如何自動處理 Network 建立、Container 啟動、Log 彙整等繁瑣工作，讓本地開發環境的切換變得極其高效。

---

## 核心觀念與實作解析

### Docker Compose CLI 的定位

首先釐清一個重要觀念：**`docker-compose` 是獨立於 Docker CLI 的 binary**。

| 環境 | 安裝狀態 |
|------|----------|
| Docker for Mac | 內建安裝 |
| Docker for Windows | 內建安裝 |
| Docker Toolbox (Windows 7) | 內建安裝 |
| Linux | 需要額外安裝 |

> Linux 使用者需要從 `github.com/docker/compose` 下載安裝。

**重要提醒**：Compose **不是 Production-grade 工具**，主要用於 **local development 和 testing**。

---

### 兩個核心指令

#### `docker-compose up` — 一鍵啟動

這是你 90% 時間會用到的指令。它會：

1. 建立 Networks
2. 建立 Volumes
3. 拉取或建立 Images
4. 啟動所有 Containers
5. 彙整顯示所有 Log

```bash
docker-compose up
# 前景執行，Log 直接顯示在 terminal

docker-compose up -d
# 背景執行
```

#### `docker-compose down` — 一鍵清除

清理所有剛才建立的資源：

```bash
docker-compose down
# 停止並移除 Containers、Networks
# 注意：Volumes 預設不會被刪除
```

---

### 實作範例：Nginx Proxy + Apache Web Server

讓我們透過一個實際案例來理解 Compose 的運作。

**目錄結構**：
```
compose-sample-2/
├── docker-compose.yml
└── nginx.conf
```

**docker-compose.yml**：
```yaml
version: '2'

services:
  proxy:
    image: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro

  web:
    image: httpd
```

**架構說明**：
- `proxy`：Nginx 作為 Reverse Proxy
- `web`：Apache 作為後端 Web Server
- `nginx.conf`：設定 Nginx 將流量轉發到 `web` Service

---

### 執行 Compose 的過程解析

**步驟 1：啟動服務**

```bash
docker-compose up
```

輸出會顯示：
1. 建立 Network（自動建立，不需要在 YAML 中指定）
2. 建立 Containers
3. 彙整顯示兩個 Container 的 Log

> **Log 彩色顯示**：Compose 會為每個 Container 指派不同顏色，方便辨識 Log 來源。

**步驟 2：驗證運作**

開啟瀏覽器前往 `http://localhost`：
- 你會看到 "It works!" — 這是 Apache 的預設頁面
- 流量路徑：Browser → Nginx Proxy → Apache Web Server

> 重新整理頁面，Terminal 會顯示流量經過 proxy 和 web 兩個 Container 的 Log。

**步驟 3：背景執行**

```bash
# 按 Ctrl+C 停止前景執行
docker-compose up -d
```

**步驟 4：查看 Log**

```bash
docker-compose logs
# 顯示所有 Container 的 Log

docker-compose logs -f
# 持續追蹤 Log
```

---

### 常用 Compose 指令一覽

```bash
# 查看運行狀態
docker-compose ps

# 查看 Container 內的 Process
docker-compose top

# 查看所有可用指令
docker-compose --help

# 清除所有資源
docker-compose down
```

> 大多數 `docker` 指令都有對應的 `docker-compose` 版本，差別在於 Compose 版本會自動使用設定檔中的配置。

---

### Compose 自動處理的細節

#### 1. Network 自動建立

即使 YAML 中沒有指定 `networks`，Compose 也會自動建立一個專屬 Network，所有 Service 都會加入其中。

> 這就是為什麼 Nginx 可以直接用 Service 名稱 `web` 連接到 Apache。

#### 2. 命名規範

Compose 會用「目錄名稱」作為 **Project Name**，自動加在所有資源前面：

| 資源 | 命名範例 |
|------|----------|
| Network | `compose-sample-2_default` |
| Container | `compose-sample-2_proxy_1` |
| Volume | `compose-sample-2_data` |

> 這個設計避免了不同專案間的資源名稱衝突。

---

### 為什麼 Compose 能改變開發流程？

老師分享了他的觀察：

**傳統開發環境的痛點**：
- 新成員 Onboarding 需要花數小時設定環境
- 不同作業系統需要不同的安裝流程
- VM 環境厚重且難以維護
- 版本衝突難以解決

**Docker Compose 的解決方案**：

```bash
# 新成員只需要兩個指令
git clone <repo>
docker-compose up
```

> 整個開發環境瞬間就緒——Database、Web Server、Proxy 全部自動配置。

---

### Read-Only 掛載

在範例中我們看到了 `:ro` 設定：

```yaml
volumes:
  - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
```

這表示 Container 只能讀取該檔案，無法修改它。這是一個安全措施，特別是當你掛載重要的設定檔時。

---

## 💡 重點摘要

- **`docker-compose up` 是一鍵啟動的魔法：自動建立 Network、Volume、Containers。**
- **`docker-compose down` 是一鍵清除的清理工具，但預設不會刪除 Volumes。**
- **Compose 會自動為所有資源加上 Project Name 前綴，避免名稱衝突。**
- **Log 彩色顯示讓多 Container 的除錯變得容易。**
- **新成員 Onboarding 從數小時縮減為兩個指令：`git clone` + `docker-compose up`。**

---

## 🔑 關鍵字

docker-compose, CLI, Network, Project Name, Reverse Proxy
