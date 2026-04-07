# 在 Stack 中使用 Secrets

## 📝 課程概述

本單元將學習如何在 Docker Stack 的 Compose 檔案中使用 Secrets。我們會比較兩種定義方式（file vs external），並理解 Stack 如何自動管理 Secrets 的生命週期。

## 核心觀念與實作解析

### Compose 版本要求

要使用 Secrets，Compose 版本必須是 **3.1 或以上**：

```yaml
version: '3.1'
```

> `version: 3` 支援 Stack 但不支援 Secrets；`version: 3.1` 才支援 Secrets。

---

### Secrets 定義方式

**方式一：使用檔案（file）**

```yaml
version: '3.1'

services:
  psql:
    image: postgres
    secrets:
      - psql_user
      - psql_pass
    environment:
      POSTGRES_USER_FILE: /run/secrets/psql_user
      POSTGRES_PASSWORD_FILE: /run/secrets/psql_pass

secrets:
  psql_user:
    file: ./username.txt
  psql_pass:
    file: ./password.txt
```

> Stack 部署時會自動建立這些 Secrets。

---

**方式二：使用外部定義（external）**

```yaml
version: '3.1'

services:
  psql:
    image: postgres
    secrets:
      - psql_pw
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/psql_pw

secrets:
  psql_pw:
    external: true
```

> 需要先手動建立 Secret，Stack 只會引用它。

---

### 兩種方式的比較

| 方式 | 優點 | 缺點 |
|------|------|------|
| `file` | 完整管理在 Stack 中 | 檔案需存在於 Server |
| `external` | Secret 可單獨管理 | 需手動建立 Secret |

**生產環境建議**：使用 `external`，讓 Secret 的建立與 Stack 部署分離，便於安全管理。

---

### 部署 Stack with Secrets

```bash
docker stack deploy -c docker-compose.yml mydb
```

**查看建立的 Secrets**：

```bash
docker secret ls
```

輸出會顯示 Stack 自動加上前綴：

```
ID      NAME              DRIVER
xxx     mydb_psql_user
xxx     mydb_psql_pass
```

---

### Stack 移除時自動清理

```bash
docker stack rm mydb
```

Stack 移除時會一併刪除其建立的 Secrets（`file` 方式）。

> 若使用 `external`，Secret 不會被刪除，需手動 `docker secret rm`。

---

### Secrets 的進階設定（長格式）

```yaml
secrets:
  - source: psql_pw
    target: db_password    # Container 內的檔名
    uid: '103'             # 擁有者 UID
    gid: '103'             # 群組 GID
    mode: 0400             # 權限
```

> 適合非 root 用戶運行的應用，可精確控制存取權限。

---

### 本地開發的相容性

`docker-compose` 在本地運行時：

- 會模擬 Secrets 功能
- 將檔案掛載到 `/run/secrets/`
- **不提供真正的加密保護**

> 這只是為了讓開發環境能使用相同的 Compose 檔案，生產環境仍需 Swarm。

---

### 安全最佳實踐

**不要**：
- ❌ 將 Secret 檔案保留在 Server 上
- ❌ 在 Bash history 中留下密碼
- ❌ 將 Secret 檔案提交到 Git

**應該**：
- ✅ 使用 CI/CD 系統注入 Secret
- ✅ 使用 `external` 模式，手動管理 Secret
- ✅ 部署後清理暫存檔案

---

## 💡 重點摘要

- **Compose 版本需 3.1 以上才能使用 Secrets。**
- **`file` 讓 Stack 自動建立 Secrets；`external` 引用預先建立的 Secrets。**
- **Stack 移除時會自動清理以 `file` 方式建立的 Secrets。**
- **本地 docker-compose 會模擬 Secrets，但不提供加密保護。**

## 🔑 關鍵字

Secrets, Stack, Compose, external, file
