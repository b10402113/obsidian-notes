# 125 Kubernetes 中的 MongoDB 部署

## 📝 課程概述

本單元將 MongoDB 實例引入 auth service 的 Kubernetes 叢集中。我們會在 `infra/k8s/` 目錄下建立 MongoDB 的 Deployment 與 ClusterIP Service，並為 MongoDB 預設監聽的 27017 port 做設定。理解「每個微服務各自擁有獨立資料庫」的原則，是本章節的核心。

---

## 核心觀念與實作解析

### 為何需要每個 Service 有自己的 MongoDB？

回顧我們的微服務架構設計，每個 service 都會有自己私有的 MongoDB instance。這是 **「One Database Per Service」** 原則的體現——資料庫不共享，才能確保每個服務的資料邊界清晰、部署獨立。

### 建立 MongoDB Deployment

在 `infra/k8s/` 目錄下新增 `auth-mongo-deploy.yaml`，命名慣例為 `{service}-mongo-deploy.yaml`。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth-mongo
  template:
    metadata:
      labels:
        app: auth-mongo
    spec:
      containers:
        - name: auth-mongo
          image: mongo
```

這裡 `image: mongo` 指的是 **Docker Hub 官方托管的 MongoDB image**，無需自行建置。預設就會監聽 port `27017`，這是所有 MongoDB client 連線時預設使用的 port。

### 建立 ClusterIP Service

Deployment 建立的是 Pod，而要讓其他 Pod 與 MongoDB 溝通，必須透過 Service：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: auth-mongo-svc
spec:
  selector:
    app: auth-mongo
  ports:
    - name: db
      protocol: TCP
      port: 27017
      targetPort: 27017
```

> **`selector`** 告訴 Service 要管理哪些 Pod（標籤為 `app: auth-mongo` 的 Pod）。**`port: 27017`** 是 Service 對外開放的 port，**`targetPort: 27017`** 是 Pod 內部實際監聽的 port。

### 部署方式

直接執行 `skaffold dev`，Skaffold 會偵測到新建立的 yaml 檔案並自動 apply。若 Skaffold 未能自動偵測到新檔案，按 `Ctrl+C` 重新執行即可。

---

## 💡 重點摘要

- MongoDB 在 Kubernetes 中就是一個跑在 Pod 內的容器，透過 Deployment 管理生命週期。
- Service 的 `selector` 標籤必須與 Deployment 模板中的 `labels` 完全對應，否則 Service 找不到 Pod。
- Port `27017` 是 MongoDB 的預設 port，無需特別研究文件即可確認（Docker Hub MongoDB image 文件有說明）。

## 🔑 關鍵字

MongoDB, Kubernetes, Deployment, ClusterIP Service, Skaffold
