# 建立私有 Docker Registry

## 📝 課程概述

本單元實際操作如何在本地端架設私有 Docker Registry。我們將學習 Registry 的運作原理、TLS 安全性考量、如何透過 Image Tag 指定推送目標，以及使用 Volume 確保資料持久化的正確方式。

## 核心觀念與實作解析

### Registry 的本質

Registry 本質上就是一個 **HTTPS Server**，預設運行於 Port 5000。與 Docker Hub 不同的是，開源版的 Docker Registry 非常精簡，沒有 Web GUI，純粹是 Web API 與儲存系統。

### TLS 安全性原則

Docker 採取 **Secure by Default** 設計哲學：

> Docker Engine 預設不會與任何未經正確 HTTPS 加密的 Registry 通訊，確保認證資訊與傳輸內容都經過加密。

**在本機測試時的例外**：若 Registry 運行於 `localhost`，Docker 允許使用 HTTP 連線，這是唯一不需要 TLS 的情境。

若需使用自簽憑證或遠端無 TLS Registry，必須在 Docker Engine 中啟用 `insecure-registry` 選項。這在正式環境中**不建議**使用。

### 實作：啟動 Registry

```bash
# 啟動 Registry Container
docker container run -d -p 5000:5000 --name registry registry

# 確認運行狀態
docker container ls
```

### Image Tag 與 Registry 指定

Image Tag 決定了 Image 要推送到哪個 Registry：

| Tag 格式 | 目標 Registry |
|----------|---------------|
| `nginx` | Docker Hub (預設) |
| `127.0.0.1:5000/nginx` | 本地 Registry |
| `myregistry.com:5000/app` | 自訂 Registry |

**操作流程**：

```bash
# 1. Pull 測試用 Image
docker pull hello-world

# 2. 重新 Tag，指定目標 Registry
docker tag hello-world 127.0.0.1:5000/hello-world

# 3. 推送到本地 Registry
docker push 127.0.0.1:5000/hello-world

# 4. 刪除本地 Image
docker image remove hello-world
docker image remove 127.0.0.1:5000/hello-world

# 5. 從本地 Registry Pull
docker pull 127.0.0.1:5000/hello-world
```

### 使用 Volume 確保資料持久化

**這是生產環境最重要的設定**。Registry 預設將資料儲存於 Container 內部，若 Container 被刪除，所有 Image 都會消失。

```bash
# 先清理舊的 Registry
docker container kill registry
docker container rm registry

# 使用 Volume 重新啟動
docker container run -d \
  -p 5000:5000 \
  --name registry \
  -v $(pwd)/registry-data:/var/lib/registry \
  registry
```

> Registry 將 Image 儲存於 `/var/lib/registry` 目錄，使用 Volume 將此目錄映射到主機可確保資料持久化。

### Registry 資料結構

透過 `tree` 指令可觀察 Registry 的儲存結構：

```
registry-data/
├── blobs/          # Image 的實際二進位資料
├── manifest/       # Image 的詮釋資料
└── repositories/   # Repository 索引
```

所有資料都使用 SHA Hash 作為目錄名稱，因為 Image 可能包含多個 Layer，這種結構能有效管理重複的 Layer。

### 正式環境的下一步

本單元展示了基本運作，但正式環境還需要：

1. **TLS 憑證設定**：使用正式簽發的憑證
2. **認證機制**：啟用 Basic Authentication
3. **儲存後端**：考慮使用 S3 或其他雲端儲存

## 💡 重點摘要

- Registry 本質是 HTTPS Server，預設 Port 5000，需注意 Docker Engine 的 TLS 要求
- Image Tag 格式決定推送目標，包含 IP:Port 即可指定自訂 Registry
- localhost 是唯一允許 HTTP 連線的例外情境
- 使用 Volume 映射 `/var/lib/registry` 是資料持久化的關鍵
- 正式環境需額外設定 TLS 憑證與認證機制

## 🔑 關鍵字

Registry, TLS, Image Tag, Volume, Port 5000
