# Swarm Secrets 安全儲存機制

## 📝 課程概述

本單元介紹 Docker Swarm 的 Secrets 功能——這是一個內建的安全儲存解決方案，用於管理密碼、API Key 等敏感資訊。我們將學習 Secrets 的運作原理、建立方式，以及如何在 Service 中使用。

## 核心觀念與實作解析

### 什麼是 Secret？

Secret 是任何「你不希望出現在報紙頭版」的敏感資訊：

- 使用者名稱 / 密碼
- TLS 憑證 / 私鑰
- SSH Key
- API Key（Twitter、AWS 等）
- 資料庫連線字串

> 如果洩漏後需要更改，那就是 Secret。

---

### Swarm Secrets 的優勢

| 特性 | 說明 |
|------|------|
| **內建整合** | 無需額外安裝 Vault 等工具 |
| **加密儲存** | Swarm Raft DB 預設加密 |
| **加密傳輸** | 使用 TLS + PKI 認證 |
| **記憶體駐留** | 不寫入磁碟（使用 tmpfs） |
| **最小權限** | 只有授權的 Container 可存取 |

---

### Secrets 的運作流程

```
┌─────────────────────────────────────────────────────────────┐
│                     Manager Node                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │            Swarm Raft Database (Encrypted)          │   │
│  │                                                     │   │
│  │   ┌─────────┐  ┌─────────┐                         │   │
│  │   │ Secret1 │  │ Secret2 │                         │   │
│  │   └────┬────┘  └────┬────┘                         │   │
│  └────────┼─────────────┼──────────────────────────────┘   │
│           │             │                                   │
│           │  TLS + PKI │                                   │
│           ▼             ▼                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   Worker Node                        │   │
│  │   ┌─────────────────────────────────────────────┐   │   │
│  │   │  Container (tmpfs: /run/secrets/)           │   │   │
│  │   │  ┌───────────┐  ┌───────────┐               │   │   │
│  │   │  │ secret1   │  │ secret2   │               │   │   │
│  │   │  └───────────┘  └───────────┘               │   │   │
│  │   └─────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

### 建立 Secret

**方式一：從檔案建立**

```bash
echo "myuser" > username.txt
docker secret create psql_user username.txt
```

**方式二：從命令行輸入**

```bash
echo "mypassword" | docker secret create psql_pass -
```

> 注意：`-` 表示從標準輸入讀取。

**查看 Secrets**：

```bash
docker secret ls
```

> 注意：`docker secret inspect` **不會**顯示 Secret 內容，只能看到 metadata。

---

### 將 Secret 指派給 Service

```bash
docker service create --name psql \
  --secret psql_user \
  --secret psql_pass \
  -e POSTGRES_USER_FILE=/run/secrets/psql_user \
  -e POSTGRES_PASSWORD_FILE=/run/secrets/psql_pass \
  postgres
```

**關鍵環境變數**：

官方 Docker Hub Image 支援 `_FILE` 後綴的環境變數，會自動讀取檔案內容：

| 環境變數 | 原本用途 | `_FILE` 版本 |
|---------|---------|-------------|
| `POSTGRES_USER` | 直接設定使用者 | 讀取 `/run/secrets/psql_user` |
| `POSTGRES_PASSWORD` | 直接設定密碼 | 讀取 `/run/secrets/psql_pass` |

---

### 驗證 Secret 在 Container 內

```bash
# 找出運行的 Container
docker service ps psql

# 進入 Container
docker exec -it <container_id> bash

# 查看 Secrets
ls /run/secrets
# 輸出：psql_user  psql_pass

# 讀取 Secret
cat /run/secrets/psql_user
# 輸出：myuser
```

---

### Secret 的限制

| 限制 | 說明 |
|------|------|
| **大小限制** | 最大 500KB |
| **僅限 Swarm** | 需要 Swarm 環境 |
| **不可變更** | 修改 Secret 需重建 Container |
| **需要 Image 支援** | Image 需支援 `_FILE` 環境變數 |

---

### 新增/移除 Secret

**移除 Secret**：

```bash
docker service update --secret-rm psql_pass psql
```

> 這會重建 Container，因為 Service 設定是不可變的（immutable）。

**新增 Secret**：

```bash
docker service update --secret-add new_secret psql
```

---

## 💡 重點摘要

- **Secrets 在 Swarm 中加密儲存、加密傳輸，僅存於記憶體（tmpfs）。**
- **使用 `_FILE` 後綴的環境變數讓 Image 能讀取 Secret 檔案。**
- **Secret 存放在 Container 的 `/run/secrets/` 目錄下。**
- **修改 Service 的 Secret 會導致 Container 重建。**

## 🔑 關鍵字

Secrets, Swarm, tmpfs, Encryption, Environment
