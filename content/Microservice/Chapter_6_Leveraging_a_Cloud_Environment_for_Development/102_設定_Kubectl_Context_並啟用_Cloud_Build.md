# 設定 kubectl Context 並啟用 Google Cloud Build

## 📝 課程概述

本單元繼續上一節的設定，透過 `gcloud container clusters get-credentials` 命令讓 kubectl 連線到 Google Cloud 上的 GKE 叢集，並啟用 **Google Cloud Build** 服務。完成後，你將能透過 Docker Desktop 的 Kubernetes 選單在本地與遠端叢集之間**自由切換**。

---

## 核心觀念與實作解析

### 兩種 Context 切換策略

根據你是否還需要在本地运行 Docker，有兩種不同的設定路徑：

#### 路徑 A：完全放棄本地 Docker（適用於確定不再需要本地叢集的學員）

```bash
# 1. 關閉 Docker Desktop
# 2. 安裝 kubectl 獨立的 client
gcloud components install kubectl

# 3. 取得 GCP 叢集的 credentials
gcloud container clusters get-credentials <叢集名稱>
```

這個方式的優點是未來可以**完全移除 Docker Desktop**，節省系統資源。

#### 路徑 B：同時保留本地 Docker（大多數學員適用）

如果你仍然需要在本地运行 Docker，只需要執行：

```bash
gcloud container clusters get-credentials ticketing-dev
```

> **原則**：除非你有明確的磁碟空間限制或確定不再需要本地叢集，否則都建議走路徑 B。

---

### 驗證 Context 切換是否成功

完成命令後，打開 **Docker Desktop → Kubernetes 選單**，你應該會看到**兩個** Context 項目：

- `docker-desktop` — 連線本地叢集
- `gke_<專案名>_<區域>_<叢集名>` — 連線 Google Cloud 遠端叢集

在終端機執行 `kubectl get pods`，`kubectl` 會使用**目前選取的 Context** 對應的叢集。你可以實際測試：切換到 `docker-desktop`，執行 `kubectl get pods`，確認看到本地叢集的 auth Pod；再切換到 GKE Context，執行同樣命令，確認看到 `No resources found`（因為遠端叢集尚未部署任何資源）。

**這個切換機制讓我們可以在本地與遠端之間隨時來回，非常方便。**

---

### 啟用 Google Cloud Build

在設定 Skaffold 之前，必須先在 Google Cloud 上啟用 Cloud Build 服務：

1. Google Cloud Console 左側選單 → **Tools** → **Cloud Build**
2. 點擊 **Enable** 啟用 API

這個步驟只需要做一次，Cloud Build API 啟用後，後續所有專案都可以使用。

---

### 更新 Skaffold 設定檔

接下來要修改 `skaffold.yaml`，告訴 Skaffold 未來使用 Google Cloud Build 而非本地 Docker 來建置 Image。

#### 修改 1：新增 Google Cloud Build 區塊

在 `build` 區塊中，在 `local: push: false` 之後**新增**以下內容：

```yaml
build:
  local:
    push: false           # 保留不刪（下一步才註釋）
  googleCloudBuild:
    projectId: <你的 GCP 專案 ID>
```

> **注意**：`projectId` 不是你在 Console 左上角看到的顯示名稱（例如 `ticketing-dev`），而是在專案設定頁面右側欄位中隨機產生的 **Project ID**（格式如 `ticketing-dev-12345`）。請至 Console 首頁的專案資訊卡中複製正確的 ID。

#### 修改 2：更新 Image 名稱格式

Skaffold 建置 Image 時，不再使用 `Docker Hub`，而是將 Image 存放到 **Google Container Registry (GCR)**。

GCR 的 Image 命名格式**固定**為：

```
us.gcr.io/<projectId>/<service-name>
```

例如：
```
us.gcr.io/ticketing-dev-12345/auth
```

你需要將這個格式同時更新到 `skaffold.yaml` 的 `build.artifacts` 區塊中每一個 Service 的 Image 欄位。

---

## 💡 重點摘要

- **`gcloud container clusters get-credentials` 是讓 kubectl 連線到 GKE 叢集的標準命令。**
- **透過 Docker Desktop 的 Kubernetes 選單即可在本地與遠端 Context 之間一鍵切換，無需修改任何設定檔。**
- **Cloud Build API 需在 Console 中手動啟用，啟用後所有專案皆可使用。**
- **`projectId` 是 GCP 自動產生的隨機 ID，而非你在 Console 看到的顯示名稱。**
- **GCR Image 的 URL 格式固定為 `us.gcr.io/<projectId>/<service-name>`，使用前需先確認正確的 Project ID。**

---

## 🔑 關鍵字

kubectl Context, gcloud container clusters, Docker Desktop, Cloud Build, Google Container Registry, GCR, Project ID, Skaffold
