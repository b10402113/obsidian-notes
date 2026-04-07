# 在 Swarm 中使用 Docker Registry

## 📝 課程概述

本單元說明如何在 Docker Swarm 叢集中部署私有 Registry。我們將學習 Swarm 環境下的特殊考量、Routing Mesh 如何簡化 Registry 存取，以及 Swarm 節點間 Image 分享的重要觀念。

## 核心觀念與實作解析

### Swarm 中 Registry 的運作方式

在 Swarm 中運行 Registry 與本地端非常相似，主要差異在於：

- 使用 `docker service create` 而非 `docker run`
- 可使用 Stack File 進行部署
- Routing Mesh 讓所有節點都能存取 Registry

> Routing Mesh 的魔法：所有節點都可以透過 `127.0.0.1:5000` 存取 Registry，無需額外設定 insecure registry。

### 持久化資料的考量

**Registry 是需要持久化資料的服務**。在正式環境中，需要決定：

1. 使用哪種 Volume Driver
2. 後端儲存方案（本地、S3、NFS 等）

本單元範例使用 Play With Docker 環境進行示範。

### 實作：在 Swarm 部署 Registry

```bash
# 建立 Registry Service
docker service create --name registry --publish 5000:5000 registry

# 確認運行狀態
docker service ps registry
```

### 驗證 Registry 運作

Registry 根目錄只返回 HTTP 200，但可透過 API 查看 Repository 列表：

```bash
# 使用 curl 存取 catalog 端點
curl http://127.0.0.1:5000/v2/_catalog

# 返回 JSON 格式的 Repository 列表
{"repositories":["hello-world"]}
```

### 推送 Image 到 Swarm Registry

```bash
# Pull 測試 Image
docker pull hello-world

# Tag 並 Push
docker tag hello-world 127.0.0.1:5000/hello-world
docker push 127.0.0.1:5000/hello-world
```

### 從 Registry 部署 Service

這是 Swarm 使用 Registry 的核心場景：

```bash
# 從私有 Registry 建立 Service
docker service create \
  --name nginx \
  --publish 80:80 \
  --replicas 5 \
  127.0.0.1:5000/nginx
```

### Swarm 節點無法共享本地 Image

**這是 Swarm 最重要的觀念之一**：

> Swarm 節點之間**無法直接分享本地 Image**。所有節點都必須從 Registry Pull Image。

```
❌ 錯誤認知：Manager1 的本地 Image 可以被 Manager2 使用
✅ 正確理解：所有節點都必須從 Registry Pull Image
```

當你使用 `docker service create` 時：
1. Swarm 決定要在哪些節點運行 Container
2. 每個節點從指定的 Registry Pull Image
3. Routing Mesh 確保所有節點都能存取 Registry

### Routing Mesh 的運作原理

```
┌─────────────┐
│   Node 1    │ ──┐
│  (Registry) │   │
└─────────────┘   │     ┌─────────────┐
                  ├──▶ │ 127.0.0.1   │
┌─────────────┐   │     │ :5000      │
│   Node 2    │ ──┤     └─────────────┘
│  (Worker)   │   │            │
└─────────────┘   │            ▼
                  │     實際導向 Registry
┌─────────────┐   │     所在的節點
│   Node 3    │ ──┘
│  (Worker)   │
└─────────────┘
```

所有節點對 `127.0.0.1:5000` 的請求，都會透過 Routing Mesh 導向 Registry 運行的節點。

### ProTip：優先使用託管服務

除非有特殊需求，否則建議優先使用託管 Registry 服務：

- **Docker Hub**、**AWS ECR**、**Quay.io** 等
- 具備備援、監控、專業儲存管理
- 開源專案通常免費
- 私有 Image 費用合理

**自行架設 Registry 的適用情境**：
- 離線環境需求
- 特殊安全限制
- 法規遵循要求

## 💡 重點摘要

- Swarm 中使用 Registry 與本地類似，Routing Mesh 簡化了存取邏輯
- Swarm 節點間無法分享本地 Image，所有節點都必須從 Registry Pull
- 正式環境需妥善規劃 Volume Driver 與儲存後端
- 除非有離線或特殊安全需求，否則建議使用託管 Registry 服務
- Registry catalog API 可查看已儲存的 Repository 列表

## 🔑 關鍵字

Swarm, Registry, Routing Mesh, Service, Volume Driver
