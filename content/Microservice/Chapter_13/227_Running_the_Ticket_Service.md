# 啟動 Tickets Service — Kubernetes 部署與 Scaffold 配置

## 📝 課程概述

本節完成了 Tickets Service 的部署設定：撰寫 Kubernetes Deployment / Service YAML 檔案、更新 Scaffold 配置、更新 MongoDB 的 Kubernetes 設定，並將 Docker 映像推送到 Docker Hub。本節的目標是讓 Tickets Service 能夠在 Kubernetes 叢集中正常啟動並連線到專屬的 MongoDB 實例。

## 核心觀念與實作解析

### Kubernetes Deployment 的複製策略

由於 Tickets Service 與 Auth Service 的部署邏輯高度相似，課程選擇直接複製 `auth-depl.yaml`，然後將所有 `auth` 相關字樣置換為 `tickets`：

- Deployment 的 `metadata.name` 與 `spec.selector.matchLabels`
- Pod Template 的標籤
- 環境變數（`JWT_KEY` 仍需保留，因為驗證 JWT 需要同一把金鑰）

**為什麼 Auth Service 的 JWT_KEY 對 Tickets Service 仍然必要？** 因為 Tickets Service 自己要負責驗證請求中的 JWT — 它不可能把請求轉給 auth service 來驗證（那是同步的 blocking 操作，會破壞微服務的獨立性）。所以每個需要認證的 service 都必須持有相同的 JWT key 來自行解碼與驗證。

### Scaffold YAML 的更新

Scaffold 透過 `skaffold.yaml` 監控檔案變更並自動同步到 Kubernetes。更新方式很直覺：複製 Auth 的區塊、修改名稱與路徑 (`tickets/src`)，`sync` 區段的 `src` 目錄保持不變，因為 Tickets Service 的程式碼同樣在 `src` 下。

### MongoDB Deployment 的複製

MongoDB 的 Kubernetes Deployment / Service 配置幾乎 100% 相同，只需要將所有 `auth` 替換為 `tickets`。Service 名稱會成為 Pod 之间互相連線的 DNS 名稱（例如：`tickets-mongo-srv:27017`）。

### Docker 映像的建立與推送

**本地環境（非 Google Cloud）才需要這個步驟。** Scaffold 預設會從 Docker Hub 拉取映像來啟動 Pod，所以必須先手動建立並推送一次：

```bash
docker build -t <docker-id>/tickets .
docker push <docker-id>/tickets
```

如果忘記這一步，Scaffold 啟動時會出現 `Cannot pull image` 錯誤，解決方法就是回到 `tickets/` 目錄重新 build 並推送。

## 💡 重點摘要

- Kubernetes Deployment 中的 `JWT_KEY` 環境變數不能刪除，因為每個 service 都要自行驗證 JWT。
- Scaffold 假設映像已經存在于 Docker Hub（本地開發流程），所以第一次部署前必須先推送。
- MongoDB 的 Kubernetes Service 名稱就是其他 Pod 連線時使用的 DNS 主機名。
- 如果映像拉取失敗，檢查 Docker Hub 是否已有正確的映像版本。

## 🔑 關鍵字

Kubernetes, Docker Hub, Scaffold, JWT_KEY, MongoDB Deployment, ClusterIP Service
