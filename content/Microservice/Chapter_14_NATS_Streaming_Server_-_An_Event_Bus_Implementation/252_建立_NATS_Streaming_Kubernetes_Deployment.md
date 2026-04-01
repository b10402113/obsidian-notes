# 建立 NATS Streaming Kubernetes Deployment

## 📝 課程概述

本單元將實作如何在 Kubernetes Cluster 中部署 NATS Streaming Server。我們會撰寫一個 Deployment 配置文件，並附加一個同時暴露 **用戶端連接埠（4222）** 與**監控埠（8222）** 的 ClusterIP Service，讓叢集內的服務能夠連線至此 Event Bus。

---

## 核心觀念與實作解析

### Deployment 配置結構

NATS Streaming Server 的 Deployment 和我們之前建立的其他 Service 沒有本質上的差異，核心結構如下：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nats-depot
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nats
  template:
    metadata:
      labels:
        app: nats
    spec:
      containers:
        - name: nats
          image: nats-streaming:0.17.0
          args:
            # 在此傳入自訂命令列參數
```

### 為什麼要指定版本 `0.17.0`？

在 Docker Image 的標籤（Tag）上，我們**不要使用 `latest`**。固定版本號可以確保每次部署的行為完全一致，避免因為 Image 更新而意外改變伺服器的行為或導致不相容。

---

### 如何傳入自訂命令列參數

Container 的 `args` 陣列，會將每個字串拆分為獨立的參數，傳給容器啟動時的主命令。格式與在 Docker CLI 中直接加參數完全相同。

**重要參數說明：**

| 參數 | 用途 |
|------|------|
| `-p 4222` | Client 連線的預設通訊埠 |
| `-m 8222` | 監控（Monitoring）HTTP 端點的埠號 |
| `-hbi 5s` | Heartbeat Interval — Server 對 Client 發送健康檢查的頻率 |
| `-hbt 5s` | Heartbeat Timeout — Client 回應 heartbeat 的超时时间 |
| `-hbpf 2` | Heartbeat Proof — Client 必須連續失敗多少次 Heartbeat 才會被認定為離線 |
| `-sc` | 設定 Store 類型（預設記憶體，可設為 `file` 或 `sql`） |
| `-sd` | 資料持久化的目錄路徑（當使用 `file` 模式時） |
| `-id ticketing` | NATS Streaming 的 Cluster ID — 所有連線至此 Server 的 Client 必須指定相同的 ID |

> 這些參數的具體意義會在後續章節中逐一揭曉。現階段只要知道：**這些設定決定了 Server 如何管理連線、事件持久化、以及如何偵測 Client 是否離線。**

---

### Service 配置：同時暴露兩個埠

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nats-srv
spec:
  selector:
    app: nats
  ports:
    - name: client
      port: 4222
      targetPort: 4222
      protocol: TCP
    - name: monitoring
      port: 8222
      targetPort: 8222
      protocol: TCP
```

我們暴露了兩個埠：

- **`client`（4222）** — 供 Service 發布與訂閱事件使用
- **`monitoring`（8222）** — HTTP 監控頁面，可查看目前所有 Client、Channel、Subscription 的狀態

---

### 部署驗證

部署完成後，可以透過 `kubectl get pods` 確認 NATS Streaming Server Pod 的狀態是否為 `Running`。如果仍處於 `ContainerCreating`，表示正在下載 Image，稍等片刻後再確認一次即可。

---

## 💡 重點摘要

- **Deployment 與 Service 的寫法與其他 Service 完全一致，沒有特殊魔法。**
- **`args` 陣列中的每個字串都會被當成獨立參數傳給容器主命令。**
- **固定使用特定版本的 Image Tag（不要用 `latest`），避免部署行為不一致。**
- **`-id ticketing` 是 Cluster ID，Client 連線時必須指定相同的 ID 才能正確連接。**

---

## 🔑 關鍵字

Deployment, ClusterIP, args, Docker Image, Cluster ID, Heartbeat, Monitoring
