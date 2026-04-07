# Compose 實戰：Drupal 多容器專案設定

## 📝 課程概述

本單元透過實際建置一個 **Drupal + PostgreSQL** 的多容器專案，練習從零撰寫 `docker-compose.yml`。我們將學習如何查閱 Docker Hub 文件、設定 Service 間的連線、使用 Named Volume 保存資料，並理解 Docker Network 如何讓 Container 間透過 Service 名稱互相通訊。

---

## 核心觀念與實作解析

### 作業目標

建立一個 Drupal CMS 網站，包含：
- **Drupal**：Content Management Server（Web 前端）
- **PostgreSQL**：後端資料庫

> 這是一個非常典型的多 Container 專案結構。

---

### 步驟 1：建立 docker-compose.yml

**基本框架**：

```yaml
version: '2'

services:
  drupal:
    image: drupal
    ports:
      - "8080:80"

  postgres:
    image: postgres
    environment:
      POSTGRES_PASSWORD: mypassword
```

---

### 步驟 2：查閱 Docker Hub 確認 Port

問題：Drupal Container 內部使用哪個 Port？

**方法一**：查看 Docker Hub 的 Dockerfile

```dockerfile
FROM php:apache
# 沒有明確的 EXPOSE，但從 php:apache 繼承
```

**方法二**：Pull Image 後用 inspect 查詢

```bash
docker pull drupal
docker image inspect drupal
# 查看 "ExposedPorts"
```

結果：Port 80

---

### 步驟 3：理解 Service 間的網路通訊

**關鍵問題**：Drupal 如何連接到 PostgreSQL？

預設情況下，Drupal 安裝精靈會詢問 Database Host，預設值是 `localhost`。但在 Docker 環境中：

- `localhost` 在 Drupal Container 內指的是「該 Container 自己」
- PostgreSQL 運行在**另一個 Container** 裡

> **解決方案**：使用 **Service 名稱** 作為 Database Host。因為 Docker Compose 會自動建立 Network，並讓 Service 名稱成為可解析的 DNS 名稱。

在 Drupal 安裝精靈中：
- Database Host 填入：`postgres`（這是我們在 YAML 中定義的 Service 名稱）

---

### 步驟 4：加入 Named Volume（額外練習）

為了讓 Drupal 的上傳檔案、模組、主題等資料能夠持久保存：

```yaml
version: '2'

services:
  drupal:
    image: drupal
    ports:
      - "8080:80"
    volumes:
      - drupal-modules:/var/www/html/modules
      - drupal-profiles:/var/www/html/profiles
      - drupal-sites:/var/www/html/sites
      - drupal-themes:/var/www/html/themes

  postgres:
    image: postgres
    environment:
      POSTGRES_PASSWORD: mypassword

volumes:
  drupal-modules:
  drupal-profiles:
  drupal-sites:
  drupal-themes:
```

> 這些 Volume 路徑來自 Docker Hub 的 Drupal 文件。

---

### 步驟 5：啟動並完成安裝

```bash
docker-compose up -d
```

開啟瀏覽器前往 `http://localhost:8080`：

1. 選擇安裝語言
2. 選擇安裝類型：Standard
3. 設定 Database：
   - Database name：`postgres`（預設）
   - Database username：`postgres`（預設）
   - Database password：`mypassword`（你在 YAML 中設定的）
   - **Advanced Options → Database host**：`postgres`（Service 名稱）
4. 完成網站設定

---

### 步驟 6：清理資源

```bash
# 停止並移除 Containers、Networks
docker-compose down

# 同時移除 Volumes
docker-compose down -v
```

> 預設 `docker-compose down` **不會刪除 Volumes**，這是 Docker 保護資料的設計。加上 `-v` 才會一併刪除。

---

### 關鍵觀念回顧

#### Service 名稱 = DNS 名稱

這是 Docker Compose 最強大的功能之一：

```yaml
services:
  drupal:
    # 可以用 "postgres" 這個名稱連接到 postgres service

  postgres:
    # 可以用 "drupal" 這個名稱連接到 drupal service
```

> 兩個 Service 自動加入同一個 Network，名稱互相解析。

#### Named Volume 的宣告

在 `services` 區塊中使用 Named Volume 後，必須在 `volumes` 區塊中宣告：

```yaml
volumes:
  drupal-modules:   # 宣告這個 Named Volume
  drupal-profiles:
```

如果不宣告，Compose 會報錯。

---

### 為什麼這個作業很重要？

老師特別強調：這是一個「每天都會遇到」的場景。

- 你不一定是開發者——可能是 SysAdmin、軟體測試人員、或想學習某個 CMS 的使用者
- 以前要在本機架設 Drupal，需要安裝 PHP、Apache、PostgreSQL、處理版本衝突......
- 現在：一個 `docker-compose.yml` 檔案 + `docker-compose up` 就搞定

> Docker Compose 大幅降低了「嘗試新軟體」的門檻。

---

## 💡 重點摘要

- **Service 名稱會自動成為 Docker Network 內的 DNS 名稱，Container 間可直接用名稱連線。**
- **查閱 Docker Hub 的 Dockerfile 或用 `docker image inspect` 確認 Container 內部 Port。**
- **Named Volume 需要在 `volumes` 區塊中宣告才能使用。**
- **`docker-compose down -v` 會一併刪除 Volumes，預設行為會保留資料。**
- **Compose 讓「試用複雜軟體」變得極其簡單——一個 YAML 檔案取代繁瑣的安裝流程。**

---

## 🔑 關鍵字

Drupal, PostgreSQL, Named Volume, DNS, Service Name
