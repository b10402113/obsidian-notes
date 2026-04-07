# 本地開發環境的 Secrets 管理

## 📝 課程概述

本單元探討如何在本地開發環境（Local Development）中使用 Docker Secrets。由於 Swarm 是生產環境工具，本地開發通常使用 Docker Compose，因此 Docker 提供了一套變通方案，讓開發者能以相同的方式處理 Secrets，實現開發與生產環境的一致性。

---

## 核心觀念與實作解析

### 本地開發的 Secrets 挑戰

在 Swarm 環境中，我們透過 `docker secret create` 將敏感資訊存入 Swarm 的加密資料庫。但在本地開發時：

- **沒有 Swarm Manager**：`docker node ls` 會顯示 "This is not a swarm manager"
- **無法存取 Swarm 資料庫**：無法使用 `docker secret` 相關命令
- **需要相同的配置方式**：為了讓開發與生產環境一致

### Docker Compose 的變通方案

當使用 `docker compose up` 時，Docker Compose 會自動處理 file-based secrets：

```bash
docker compose up -d
docker compose exec psql cat /run/secrets/psql_user
```

**背後的運作機制**：

> Docker Compose 實際上是將本地檔案以 Bind Mount 的方式掛載到容器中，類似於執行 `-v` 參數。

**重要提醒**：
- 這個機制**完全不安全**（totally not secure），這是刻意的設計
- 目的是讓開發者能用相同的流程和環境變數來處理 Secrets
- **需要 Docker Compose v1.11 以上版本**才支援此功能

### File-based vs External Secrets

| 類型 | Swarm 生產環境 | 本地開發環境 |
|------|---------------|-------------|
| File-based Secrets | ✅ 支援 | ✅ 支援 |
| External Secrets | ✅ 支援 | ❌ 不支援 |

如果生產環境使用 External Secrets，開發環境需要準備另一個 Compose 檔案，使用 file-based secrets 並指定範例密碼檔案。

---

### Compose 檔案組織：完整的應用程式生命週期

為了實現「從開發到生產的一致性」，可以組織多個 Compose 檔案：

#### 1. 基礎配置檔（docker-compose.yml）

定義所有環境共用的預設值：
- 服務名稱與 images
- 基本的服務定義

#### 2. 本地開發覆蓋檔（docker-compose.override.yml）

這個檔案會被 Docker Compose **自動載入**：

```yaml
services:
  drupal:
    build: .
    ports:
      - "8080:80"
    volumes:
      - ./themes:/var/www/html/themes
```

特色：
- **自動合併**：執行 `docker compose up` 時自動套用
- 本地建置 image（而非使用 registry image）
- 設定 Bind Mounts 方便即時開發

#### 3. CI 環境配置（docker-compose.test.yml）

用於 Jenkins、Codeship 等 CI 平台：

```bash
docker compose -f docker-compose.yml -f docker-compose.test.yml up -d
```

特色：
- 每次建置新 image
- 可能使用範例資料庫 mount
- 不需要持久化資料

#### 4. 生產環境配置（docker-compose.prod.yml）

用於 Stack Deploy：

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml config > output.yml
docker stack deploy -c output.yml myapp
```

特色：
- 使用 External Secrets
- 正確的 Volume 配置
- 生產等級的設定

---

### docker compose config 命令

這個命令可以將多個 Compose 檔案合併輸出：

```bash
docker compose -f docker-compose.yml -f docker-compose.prod.yml config
```

**注意事項**：
- 早期版本有 bug，可能不會列出 secrets（需檢查輸出）
- `extends` 選項目前在 Swarm Stacks 中尚未完全支援

---

## 💡 重點摘要

- **Docker Compose 透過 Bind Mount 模擬 Secrets 功能，讓本地開發能使用與生產環境相同的配置流程。**
- **File-based secrets 在本地開發可直接運作；External secrets 需要為開發環境準備替代配置。**
- **使用多個 Compose 檔案搭配 `-f` 參數，可管理不同環境的配置差異。**
- **`docker compose config` 可合併多個配置檔，方便產生生產環境的 Stack 檔案。**
- **目標是讓開發、測試、生產環境使用相同的 image 和配置流程。**

---

## 🔑 關鍵字

Docker Compose, Secrets, Swarm, Stack, Bind Mount, Override
