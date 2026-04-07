# Image Tagging 與上傳 Docker Hub

## 📝 課程概述

本單元詳細說明 Docker Image 的 Tagging 機制，以及如何將 Image 上傳到 Docker Hub。理解 Tag 的命名規則與 Registry 的運作方式，是分享與分發 Container Image 的必備技能。

---

## 核心觀念與實作解析

### Image 的識別方式

Image 本身沒有「名稱」，我們透過 **Repository + Tag** 來識別：

```
bretfisher/nginx:latest
    │           │
    │           └── Tag（標籤）
    └────────────── Repository（包含帳號/組織名稱）
```

#### Repository 命名規則

| 類型 | 格式 | 範例 |
|------|------|------|
| **官方 Image** | `<image_name>` | `nginx`、`mysql` |
| **個人/組織 Image** | `<account>/<image_name>` | `bretfisher/nginx` |
| **組織 Image** | `<org>/<image_name>` | `mysql/mysql-server` |

> **只有官方 Image 可以不帶帳號/組織名稱。**

---

### Tag 的本質

#### Tag 是指向 Image ID 的指標

```
Image ID: sha256:abc123...

        ┌──────────────┐
        │   nginx      │
        │   latest     │──┐
        └──────────────┘  │
                          │
        ┌──────────────┐  │    同一個 Image ID
        │   nginx      │  │
        │   mainline   │──┼──► sha256:abc123...
        └──────────────┘  │
                          │
        ┌──────────────┐  │
        │   nginx      │  │
        │   1.11.9     │──┘
        └──────────────┘
```

**重要觀念：**
- 同一個 Image 可以有多個 Tag
- 不同 Tag 指向同一個 Image ID 時，實際上是同一份資料
- Tag 類似 Git 的 tag，是指向特定 commit 的指標

#### `latest` Tag 的陷阱

```bash
docker pull nginx  # 等同於 docker pull nginx:latest
```

> **`latest` 只是預設 Tag，不代表「最新版本」。** 你可以把任何 Image 標記為 `latest`。

**最佳實踐：** 在 Docker Hub 的官方 Image 中，`latest` 通常確實指向最新穩定版，但不應過度依賴這個假設。

---

### Image Tag 操作

#### 新增 Tag

```bash
docker image tag <source_image> <new_image:tag>
```

**範例：**
```bash
# 為現有 nginx Image 新增 Tag
docker image tag nginx bretfisher/nginx

# 新增帶版本號的 Tag
docker image tag nginx bretfisher/nginx:testing
```

執行後，`docker image ls` 會顯示多個名稱，但它們的 Image ID 相同。

---

### Docker Hub 登入與認證

#### 登入 Docker Hub

```bash
docker login
```

登入成功後，認證資訊會儲存在 `~/.docker/config.json`。

> **安全提醒：** 在共用機器上，使用完畢後應執行 `docker logout` 清除認證資訊。

---

### 上傳 Image 到 Docker Hub

#### Push Image

```bash
docker image push <account>/<image_name>:<tag>
```

**範例：**
```bash
docker image push bretfisher/nginx:latest
docker image push bretfisher/nginx:testing
```

#### Layer 已存在的優化

當 Push 的 Image Layer 已存在於 Docker Hub 時：

```
$ docker image push bretfisher/nginx:testing
The push refers to repository [docker.io/bretfisher/nginx]
5f70bf18a086: Layer already exists    ← Layer 已存在，跳過
bcff5a6a793e: Layer already exists
...
```

> **這是 Layer 架構的優勢：** 只傳輸不存在的 Layer，節省頻寬與時間。

---

### 私有 Repository

若要建立私有 Image：

1. **先在 Docker Hub 建立 Private Repository**
2. 再 Push Image 到該 Repository

> 如果直接 Push 到不存在的 Repository，預設會建立公開 Repository。

---

## 💡 重點摘要

- **Image 透過 Repository + Tag 識別；官方 Image 不需要帳號前綴，個人/組織 Image 需要。**
- **Tag 是指向 Image ID 的指標，同一個 Image 可以有多個 Tag。**
- **`latest` 只是預設 Tag 名稱，不保證是最新版本。**
- **Push Image 前需先 `docker login`，認證資訊儲存在 `~/.docker/config.json`。**
- **Layer 已存在時會跳過傳輸，這是 Docker 分層架構的優勢。**

---

## 🔑 關鍵字

Tag, Repository, Docker Hub, Registry, Push, Login, Image ID
