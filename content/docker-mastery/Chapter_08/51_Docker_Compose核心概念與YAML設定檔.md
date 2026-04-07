# Docker Compose 核心概念與 YAML 設定檔

## 📝 課程概述

本單元正式介紹 **Docker Compose**——一個能夠大幅簡化多 Container 應用管理的工具。我們將學習 Docker Compose 的兩大組成部分：YAML 設定檔與 CLI 工具，以及如何透過設定檔將原本複雜的 `docker run` 指令轉化為可讀性高、易於維護的宣告式配置。

---

## 核心觀念與實作解析

### 為什麼需要 Docker Compose？

在真實世界中，**極少數服務是真正獨立運作的**。Container 的設計是「單一 Process」，但實際應用往往需要多個 Container 協作：

- **Database**：PostgreSQL、MySQL、Redis 等
- **Web Server**：Nginx、Apache 等
- **Application Server**：Backend API、Worker 等
- **Proxy**：Load Balancer、Reverse Proxy 等

> 問題來了：每次啟動這些服務，我們需要記住所有的 `docker run` 選項、設定 Network、管理 Port 映射、處理 Environment Variables......這很快就會變成一場惡夢。

**Docker Compose 解決了這個問題**：

- 用一個 YAML 檔案定義所有 Container、Network、Volume
- 一個指令啟動所有服務
- 一個指令清除所有資源
- 自動建立 Container 間的 Network 連線

---

### Docker Compose 的兩個組成部分

#### 1. YAML 設定檔（`docker-compose.yml`）

這是宣告式的配置檔案，用來定義：
- **Services**：要運行的 Container
- **Networks**：Container 間的網路設定
- **Volumes**：持久化儲存

> YAML 是一種非常容易理解的配置語言，比 INI 檔案更容易閱讀，因為它支援層級結構。

#### 2. CLI 工具（`docker-compose`）

用來執行 YAML 設定的命令列工具：
- `docker-compose up`：啟動所有服務
- `docker-compose down`：停止並清除所有資源

> 主要用途是 **local dev 和 test**，讓本地開發環境的設定變得極其簡單。

---

### Compose File 的版本演進

Docker Compose 最初叫做 **Fig**，當時沒有版本號碼。隨著功能增加，現在需要在檔案開頭指定版本：

```yaml
version: '2'
```

| 版本 | 說明 |
|------|------|
| 1 | 預設版本（不建議使用，功能受限） |
| 2 | 建議的最小版本，支援 Networks、Volumes |
| 3.x | 支援 Swarm 部署（2017 年起，可用於 Production） |

> 老師建議：**至少使用 version 2**，除非需要 Swarm 的特定功能才用 version 3。

---

### Compose File 的基本結構

```yaml
version: '2'

services:
  # Container 定義
  web:
    image: nginx
    ports:
      - "8080:80"

  database:
    image: postgres
    environment:
      POSTGRES_PASSWORD: example

volumes:
  # Named Volume 定義（可選）

networks:
  # 自訂 Network 定義（可選）
```

**三大區塊**：
- `services`：Container 定義（必須）
- `volumes`：Named Volume 定義（可選）
- `networks`：自訂 Network 定義（可選）

---

### Services 與 Container 的關係

為什麼叫 **Services** 而不是 Containers？

> 因為一個 Service 可以有多個相同的 Container（用於 Redundancy 或 Scaling）。「Service」更精確地描述了「提供某種功能的一組 Container」。

**Service 名稱的重要意義**：
- 可以自由命名（不需要與 Image 名稱相關）
- **會成為 Docker Network 內的 DNS 名稱**
- 類似 `docker run --name` 的效果

---

### YAML 語法重點

#### Key-Value 格式（單一值）

當一個設定只有一個值時：

```yaml
image: nginx
```

#### List 格式（多個值）

當設定可以有多個值時，使用 `-` 加上縮排：

```yaml
ports:
  - "8080:80"
  - "443:443"

volumes:
  - /host/path:/container/path
```

> 注意：複數名稱（`ports`、`volumes`）通常代表這是 List 格式。

#### Environment Variables 的兩種寫法

**List 格式**：
```yaml
environment:
  - POSTGRES_PASSWORD=example
  - POSTGRES_USER=admin
```

**Key-Value 格式**：
```yaml
environment:
  POSTGRES_PASSWORD: example
  POSTGRES_USER: admin
```

---

### 實際範例：Jekyll 靜態網站

```yaml
version: '2'

services:
  jekyll:
    image: bretfisher/jekyll-serve
    volumes:
      - .:/site
    ports:
      - "80:4000"
```

重點：
- `.` 代表當前目錄（Compose 專用語法，等同於 `$(pwd)`）
- 這個設定等同於之前學過的 `docker run` 指令

---

### 實際範例：WordPress + PostgreSQL

```yaml
version: '2'

services:
  wordpress:
    image: wordpress
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: database
      WORDPRESS_DB_PASSWORD: example

  database:
    image: postgres
    environment:
      POSTGRES_PASSWORD: example
```

重點：
- `WORDPRESS_DB_HOST: database` — 這裡的 `database` 是 Service 名稱，也是 DNS 名稱
- 兩個 Service 自動加入同一個 Network，可以互相通訊

---

### 進階範例：depends_on 設定

```yaml
services:
  ghost:
    image: ghost
    depends_on:
      - db_proxy

  db_proxy:
    image: mysql-proxy
    depends_on:
      - mysql1
      - mysql2

  mysql1:
    image: mysql

  mysql2:
    image: mysql
```

**`depends_on` 的作用**：
- 告訴 Compose Service 之間的啟動順序
- 確保被依賴的 Service 先啟動

---

### 參考文件

老師強烈建議將以下資源加入書籤：

- **Docker Compose File 官方文件**：`docs.docker.com`
- 搜尋關鍵字：「Docker Compose file」

> 「我每天都會查閱這份文件，因為功能太多了，沒人能全部記住。」

---

## 💡 重點摘要

- **Docker Compose 將複雜的 docker run 指令轉化為可讀性高的 YAML 設定檔。**
- **Compose 包含兩部分：YAML 設定檔（定義服務）與 CLI 工具（執行設定）。**
- **Service 名稱會成為 Docker Network 內的 DNS 名稱，讓 Container 間可以互相找到對方。**
- **建議至少使用 version 2，以支援 Networks 和 Volumes 功能。**
- **官方文件是最好的參考來源，隨時查閱是正常的做法。**

---

## 🔑 關鍵字

Docker Compose, YAML, Services, Networks, depends_on
