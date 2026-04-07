# Compose 進階：Dockerfile 與自訂 Image 整合

## 📝 課程概述

本單元延續 Drupal 專案，進一步學習如何在 Compose 中整合自訂 Dockerfile。我們將建立一個包含預設 Theme 的客製化 Drupal Image，並理解「Build + Image」並存時 YAML 設定的特殊行為，以及如何透過 Volume 讓資料在 Compose 重啟後依然保留。

---

## 核心觀念與實作解析

### 作業目標

在前一個作業的基礎上，我們要：
1. 建立自訂 Dockerfile，將 Bootstrap Theme 預先安裝進 Drupal Image
2. 修改 `docker-compose.yml`，讓它使用我們建置的自訂 Image
3. 確保 PostgreSQL 資料能夠持久保存

---

### Dockerfile 撰寫

```dockerfile
FROM drupal:8.2

# 安裝 Git（用來下載 Theme）
RUN apt-get update && apt-get install -y git \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /var/cache/apt/*

# 切換到 themes 目錄
WORKDIR /var/www/html/themes

# 下載 Bootstrap Theme（最佳化：只取最新 commit，不取完整歷史）
RUN git clone --branch 8.x-3.x --depth 1 https://git.drupal.org/project/bootstrap.git

# 修正權限（Apache 以 www-data 身份執行）
RUN chown -R www-data:www-data /var/www/html/themes

# 回到預設工作目錄
WORKDIR /var/www/html
```

---

### Dockerfile 關鍵技巧解析

#### 1. `apt-get` 的清理慣例

```dockerfile
RUN apt-get update && apt-get install -y git \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /var/cache/apt/*
```

為什麼要這樣做？

- `apt-get update` 會在 `/var/lib/apt/lists/` 建立約 10MB 的 Cache
- 這些檔案在 Image 運行時不需要
- 用一個 `RUN` 指令完成「安裝 + 清理」，可以減少 Image Layer 大小

> 這是 Docker Image 最佳實踐的標準模式。

#### 2. Git Clone 的最佳化

```dockerfile
RUN git clone --branch 8.x-3.x --depth 1 https://git.drupal.org/project/bootstrap.git
```

| 參數 | 說明 |
|------|------|
| `--branch 8.x-3.x` | 只下載特定分支 |
| `--depth 1` | 只下載最新的 1 個 commit |

> 這樣可以節省大量時間和空間——我們只需要最新版本的檔案，不需要完整的 Git 歷史。

#### 3. 權限修正

Dockerfile 中的 `RUN` 指令預設以 `root` 身份執行。Git Clone 下來的檔案會是 `root` 擁有。

但 Drupal（基於 Apache）以 `www-data` 身份執行，需要有權限存取這些檔案。

```dockerfile
RUN chown -R www-data:www-data /var/www/html/themes
```

> 這是 Docker 中常見的權限問題，務必注意。

---

### 修改 docker-compose.yml

**關鍵變更**：加入 `build` 設定

```yaml
version: '2'

services:
  drupal:
    build: .
    image: custom-drupal
    ports:
      - "8080:80"

  postgres:
    image: postgres
    environment:
      POSTGRES_PASSWORD: mypassword
    volumes:
      - drupal-data:/var/lib/postgresql/data

volumes:
  drupal-data:
```

---

### `build` + `image` 並存時的特殊行為

當 Service 同時有 `build` 和 `image` 設定時：

```yaml
services:
  drupal:
    build: .
    image: custom-drupal
```

**行為改變**：

| 情況 | `image` 的意義 |
|------|---------------|
| 只有 `image` | 從 Docker Hub 拉取這個 Image |
| `build` + `image` | 用 `build` 的設定建置 Image，並命名為 `image` 指定的名稱 |

> 這讓你能夠用友善的名稱來識別自訂建置的 Image。

---

### `build: .` 簡寫語法

```yaml
build: .
```

等同於：

```yaml
build:
  context: .
  dockerfile: Dockerfile
```

> 使用當前目錄作為 Build Context，並使用預設名稱 `Dockerfile`。

---

### PostgreSQL Volume 設定

為了讓資料庫內容在 Compose 重啟後保留：

```yaml
services:
  postgres:
    volumes:
      - drupal-data:/var/lib/postgresql/data

volumes:
  drupal-data:
```

> `/var/lib/postgresql/data` 是 PostgreSQL 官方 Image 的 Volume 路徑，可從 Docker Hub 查詢。

---

### 執行流程

```bash
docker-compose up
```

1. 檢查 `custom-drupal` Image 是否存在
2. 不存在 → 執行 Dockerfile Build
3. 建立 Networks、Volumes
4. 啟動 Containers

---

### 驗證 Theme 安裝成功

1. 完成 Drupal 安裝精靈
2. 進入後台 → Appearance
3. 在「Uninstalled themes」區塊會看到 Bootstrap theme
4. 點擊「Install and set as default」
5. 回到首頁，網站外觀已變更

---

### 資料持久化驗證

```bash
# 停止並移除 Containers
docker-compose down

# 重新啟動
docker-compose up

# 開啟瀏覽器 → 網站設定和內容都還在
```

> 這就是 Named Volume 的威力——Container 生命週期與資料生命週期解耦。

---

### 清理時的注意事項

```bash
# 移除 Containers、Networks、Images
docker-compose down

# Named Volume 預設不會被刪除
docker volume ls
# 仍然可以看到 drupal-data

# 如果要同時刪除 Volume
docker-compose down -v
```

> 這個設計是刻意的——Docker 假設你想要保留資料。

---

### 為什麼這個模式很有用？

老師指出這個範例展示了 Docker Compose 對「非開發者」的價值：

- **SysAdmin**：快速部署測試環境
- **QA/Tester**：建立一致的測試環境
- **軟體學習者**：快速試用複雜軟體（如 Drupal、WordPress、GitLab 等）

> 以前要在 Windows/Mac 上安裝 Drupal + PostgreSQL 可能需要數小時。現在：Clone Repo + `docker-compose up`，幾分鐘內搞定。

---

## 💡 重點摘要

- **Dockerfile 中使用 `git clone --depth 1` 可大幅減少下載時間和 Image 大小。**
- **`apt-get` 安裝後務必清理 Cache，這是 Image 最佳化的標準做法。**
- **同時有 `build` 和 `image` 時，`image` 變成「建置後的 Image 名稱」。**
- **Dockerfile 以 root 執行，需注意檔案權限是否符合 Application 需求。**
- **Named Volume 讓資料在 Compose down/up 後依然保留，是資料持久化的關鍵。**

---

## 🔑 關鍵字

Dockerfile, build, git clone, Permission, Named Volume
