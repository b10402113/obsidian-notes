# 125_Kubernetes_中的_MongoDB_資料庫部署

## 📝 課程概述

本章正式進入「使用者註冊流程」的實作階段。首先要解決的是「資料要存在哪裡」的問題——我們將在 Kubernetes cluster 內部署一個獨立的 MongoDB instance，並透過 ClusterIP Service 讓 auth service 能夠與其溝通。這是「每個 service 擁有自己資料庫」原則的具體落實。

---

## 核心觀念與實作解析

### 為什麼要在 Kubernetes 裡跑資料庫？

許多初次接觸微服務的開發者會疑惑：資料庫通常不就是一個獨立的伺服器嗎？為什麼要放在 Kubernetes 裡？

老師在這裡強調的核心原則是：**每個 microservice 應該擁有自己的資料庫實例**，彼此之間不共享。這不是技術限制，而是架構設計上的選擇——當各服務的資料庫各自獨立，服務之間的耦合度就會大幅降低。

在本地開發環境中，我們把 MongoDB 也以 Pod 的形式跑在 cluster 內，這樣可以透過同一套 Kubernetes 設定（ Deployment + Service ）來管理所有元件，不需要另外維護一台獨立的資料庫機器。

### Deployment + Service 的命名慣例

課程中使用 `auth-mongo-deploy.yaml` 作為檔案名稱，背後有一套命名邏輯：

```
{服務名稱}-{資料庫類型}-deploy.yaml
```

- `auth`：擁有這個資料庫的 service
- `mongo`：資料庫類型（MongoDB）
- `deploy`：代表這是 deployment 設定檔

在 YAML 內，Deployment 與 Service 會寫在同一個檔案中，以 `---` 分隔。

### Deployment 的設定細節

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
        app: auth-mongo   # 這會打在 Pod 上
    spec:
      containers:
        - name: auth-mongo
          image: mongo    # 來自 Docker Hub 的官方 image
```

> **重要觀念**：這裡的 label 設計是為了讓 Deployment 能找到它所管理的 Pod。`selector.matchLabels` 告訴 Deployment「哪些 Pod 歸我管」，而 `template.metadata.labels` 就是實際打在 Pod 上的標籤。兩者必須完全一致。

### Service 的設定細節

```yaml
apiVersion: v1
kind: Service
metadata:
  name: auth-mongo-svc
spec:
  selector:
    app: auth-mongo     # 找到带有此 label 的 Pod
  ports:
    - name: db
      protocol: TCP
      port: 27017       # Service 對外暴露的端口
      targetPort: 27017 # Pod 內容器監聽的端口
```

> Service 的 `selector` 負責「找到哪些 Pod」，所以這裡的 label 必須與 Deployment 的 `template.metadata.labels` 匹配。

### 關於 Port 27017

MongoDB 預設監聽在 **27017**，這不是 Kubernetes 的約定，而是 MongoDB 本身的預設值。要確認這件事，需要去 Docker Hub 查看該 image 的文件。這是部署任何非原生 Kubernetes 元件時的必要功課——**每個元件都有自己預設的端口，必須查文件確認**。

### Scaffold 的自動偵測

使用 `skaffold dev` 時，它會監控 `infra/k8s` 目錄的變化並自動 apply。但課程中提到，**Scaffold 有時不會偵測到新建立的檔案**，這時需要重啟 Scaffold（Ctrl+C 後重新執行 `skaffold dev`）。如果 apply 成功，終端會顯示 Deployment 和 Service 都被建立出來。

---

## 💡 重點摘要

- 每個 microservice 應擁有獨立的 MongoDB instance，貫徹「資料庫 per service」的設計原則
- MongoDB 在 Kubernetes 內以 Deployment + ClusterIP Service 的形式部署，命名慣例為 `{service}-{dbtype}`
- Deployment 的 `selector` 與 Pod template 的 `labels` 必須完全匹配，否則 Deployment 無法管理對應的 Pod
- 部署任何 Docker image 前，務必查詢該 image 的官方文件，確認預設監聽端口

## 🔑 關鍵字

MongoDB, Kubernetes, Deployment, ClusterIP, Docker Hub, Scaffold
