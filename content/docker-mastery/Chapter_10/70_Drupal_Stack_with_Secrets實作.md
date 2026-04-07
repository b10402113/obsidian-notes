# Drupal Stack with Secrets 實作

## 📝 課程概述

本單元是 Secrets 的整合練習，我們將把之前的 Drupal Compose 專案改造成生產級 Stack，並使用 Secrets 來保護資料庫密碼。這個練習涵蓋了從開發環境遷移到 Swarm 部署的完整流程。

## 核心觀念與實作解析

### 專案改造步驟

**原始 Compose 檔案問題**：

```yaml
version: '2'  # ❌ 需升級到 3.1

services:
  drupal:
    build: .  # ❌ Stack 不支援 build
    ...
  postgres:
    environment:
      POSTGRES_PASSWORD: mypassword  # ❌ 密碼明碼
```

---

### 步驟一：升級 Compose 版本

```yaml
version: '3.1'  # 必須是 3.1 以上
```

---

### 步驟二：移除 build，改用官方 Image

```yaml
services:
  drupal:
    image: drupal:8.2  # 使用官方 Image，固定版本
    # build: .  # 移除
```

> 生產環境應固定 Image 版本（tag），避免 `latest` 帶來的不確定性。

---

### 步驟三：改用 Secrets 管理密碼

```yaml
services:
  postgres:
    image: postgres:9.6
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/psql_pw
    secrets:
      - psql_pw
```

> 使用 `_FILE` 後綴讓 Image 從檔案讀取密碼。

---

### 步驟四：定義 external Secret

```yaml
secrets:
  psql_pw:
    external: true
```

> 表示這個 Secret 需要在部署前手動建立。

---

### 完整 Compose 檔案

```yaml
version: '3.1'

services:
  drupal:
    image: drupal:8.2
    ports:
      - "8080:80"
    volumes:
      - drupal-modules:/var/www/html/modules
      - drupal-profiles:/var/www/html/profiles
      - drupal-sites:/var/www/html/sites
      - drupal-themes:/var/www/html/themes
    depends_on:
      - postgres

  postgres:
    image: postgres:9.6
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/psql_pw
    volumes:
      - drupal-data:/var/lib/postgresql/data
    secrets:
      - psql_pw

volumes:
  drupal-modules:
  drupal-profiles:
  drupal-sites:
  drupal-themes:
  drupal-data:

secrets:
  psql_pw:
    external: true
```

---

### 部署流程

**1. 先建立 Secret**：

```bash
echo "mySecurePassword123" | docker secret create psql_pw -
```

**2. 部署 Stack**：

```bash
docker stack deploy -c docker-compose.yml drupal
```

**3. 驗證 Stack 狀態**：

```bash
docker stack ps drupal
```

---

### 常見錯誤排查

**錯誤：Secret not found**

```
failed to create service: secrets psql_pw not found
```

**原因**：忘記先建立 Secret

**解決**：先執行 `docker secret create psql_pw -`

---

**錯誤：Compose version not supported**

```
cannot use compose version 2 file
```

**原因**：版本太舊

**解決**：將 `version: '2'` 改為 `version: '3.1'`

---

### 驗證密碼正確性

在 Drupal 安裝過程中，當需要填寫資料庫連線資訊時：

| 欄位 | 值 |
|------|---|
| Database name | postgres |
| Database username | postgres |
| Database password | mySecurePassword123 |

如果連線成功，代表 Secret 正確傳遞到 Container。

---

## 💡 重點摘要

- **從開發環境遷移到 Stack 需：升級版本、移除 build、改用 Secrets。**
- **使用 `external: true` 時，需先手動建立 Secret。**
- **生產環境應固定 Image 版本，使用 `image: drupal:8.2` 而非 `latest`。**
- **部署前確認版本號與 Secret 存在，避免常見錯誤。**

## 🔑 關鍵字

Drupal, Stack, Secret, external, Image
