# Docker Hub 深入解析

## 📝 課程概述

本單元深入探討 Docker Hub 的進階功能，說明為什麼 Image Registry 是容器生態系統中不可或缺的組成部分。我們將學習 Docker Hub 的自動化建置、Webhook 整合、Repository Links 等功能，以及如何與 GitHub/BitBucket 整合實現 CI/CD 流程。

## 核心觀念與實作解析

### 為什麼需要 Image Registry？

在容器生態系統中，Image Registry 並非選用功能，而是**必要元件**。Docker 與其他容器工具的設計本質上需要一個獨立於伺服器與開發工作站之外的地方來儲存 Image。

雖然技術上可以透過 zip 檔案進行 export/import，但這種方式相當繁瑣。因此，無論是使用網際網路上的公開 Registry 或私有 Registry，都需要將其納入規劃。

### Docker Hub 核心功能

Docker Hub 是全球最受歡迎的容器 Image Registry，除了基本的儲存功能外，還提供以下進階能力：

#### Repository 權限管理

- **公開與私有 Repository**：每個帳號可免費獲得一個私有 Repository，其餘需付費
- **Collaborators 功能**：可針對每個 Image 設定其他使用者的權限
- **Organizations**：完全免費，可建立團隊並管理成員權限，類似 GitHub 的組織架構

#### Webhook 自動化

Webhook 是實現 CI/CD 的關鍵機制：

> 當 Image 建置完成並推送到 Docker Hub 後，可自動觸發 Jenkins、Codeship、Travis CI 等服務，繼續後續的自動化流程。

這讓我們能夠建立從 Git/GitHub 到 Docker，一路到伺服器的完整自動化部署管線。

### 自動化建置 (Automated Build)

**這是 Docker Hub 最強大的功能之一**。若使用 GitHub 或 BitBucket 儲存程式碼，不應直接建立 Repository，而應選擇「Create Automated Build」：

1. 連結 GitHub/BitBucket 帳號至 Docker Hub
2. 選擇來源 Repository
3. 設定觸發建置的 Branch 條件
4. 每次 commit 後自動建置 Image
5. 收到建置成功或失敗的 Email 通知

> 自動化建置是 Image 品質的指標之一，表示該 Image 持續與原始碼保持同步。

### Repository Links — 依賴更新機制

當 Dockerfile 使用 `FROM` 指令引用其他 Image 時，可以設定 Repository Links：

```
# 若你的 Image 依賴 Jekyll 官方 Image
FROM jekyll/jekyll
```

設定連結後，**當基礎 Image 被更新時，你的 Image 也會自動重建**。這對於安全性更新和錯誤修正至關重要。

### Build Triggers

除了 Git commit 觸發外，Docker Hub 還提供 Build Triggers：

- 產生唯一的 POST URL
- 讓非 GitHub 的系統也能觸發建置
- 適用於自訂 CI 系統或排程建置

### Docker Registry — 開源私有 Registry

若需自行架設私有 Registry，可使用 Docker 官方提供的開源方案：

| 特性 | 說明 |
|------|------|
| 位置 | `docker/distribution` GitHub Repository |
| 安裝 | `docker pull registry` 即可使用 |
| 預設 Port | 5000 |
| Web GUI | 無 (非常精簡) |
| 認證 | 支援 Basic Auth，但為全有或全無模式 |

> Docker Registry 的認證機制類似 Apache htaccess，取得帳密後即擁有完整的讀寫權限，不具備 RBAC (Role-Based Access Control) 功能。

### Storage Drivers 支援

Docker Registry 支援多種後端儲存方案：

- **本地儲存**：預設，使用本機檔案系統
- **雲端儲存**：S3、Azure、Alibaba、Google Cloud
- **OpenStack Swift**

### 重要資源與進階功能

1. **TLS 設定**：正式環境應啟用 HTTPS，避免 insecure registry 設定
2. **Garbage Collection**：Registry 會佔用大量儲存空間，需定期清理舊版 Image
3. **Registry Mirror**：代理模式，可快取 Docker Hub Image，減少頻寬消耗並提供容錯能力

## 💡 重點摘要

- Image Registry 是容器生態系統的必要元件，非選用功能
- Docker Hub 的 Automated Build 讓 Image 與原始碼保持同步，是品質保證的重要指標
- Repository Links 確保基礎 Image 更新時，依賴的 Image 也會自動重建
- Webhook 機制讓 Docker Hub 能與 CI/CD 工具無縫整合
- 私有 Registry 適合有離線需求或特殊安全限制的場景

## 🔑 關鍵字

Docker Hub, Registry, Automated Build, Webhook, Repository Links
