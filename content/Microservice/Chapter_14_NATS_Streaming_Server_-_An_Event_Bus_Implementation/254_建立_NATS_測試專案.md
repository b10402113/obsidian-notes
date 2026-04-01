# 建立 NATS 測試專案

## 📝 課程概述

本單元實作一個獨立的 TypeScript 子專案，用來直接與執行於 Kubernetes 內的 NATS Streaming Server 互動。我們會建立 `publisher.ts` 與 `listener.ts` 兩個獨立程式，分別負責發布事件與訂閱事件，並透過 `kubectl port-forward` 讓本地端程式能夠直連叢集內的 NATS Server。

---

## 核心觀念與實作解析

### 專案初始化

```bash
mkdir nats-test
cd nats-test
npm init -y
npm install node-nats-streaming ts-node-dev typescript @types/node
```

- **`node-nats-streaming`** — 這就是我們用來與 NATS Streaming Server 溝通的 Client Library
- **`ts-node-dev`** — TypeScript 環境下的熱重載執行器，檔案改變時自動重啟

---

### 為什麼要建立獨立測試專案？

在真正接入 Ticket Service 之前，我們需要**在隔離環境中理解 NATS Streaming 的內部行為**。如果直接在正式 Service 中測試，很多訊息傳遞的細節會被其他複雜邏輯淹沒，難以建立直覺。

---

### `publisher.ts` 的核心結構

```typescript
import nats from 'node-nats-streaming';

// 建立 Client（Stan = Streaming server 的客戶端實例）
const stan = nats.connect('ticketing', 'abc', {
  url: 'http://localhost:4222'
});

stan.on('connect', () => {
  console.log('Publisher connected to NATS');
});
```

**重要參數說明：**

1. **`'ticketing'`** — Cluster ID，必須與 NATS Server 啟動時的 `-id` 參數一致
2. **`'abc'`** — Client ID，用於在 Server 端識別這個客戶端。**同一個 Client ID 只能同時有一個連線**
3. **`url`** — NATS Server 的端點，本地端透過 `port-forward` 連接時為 `localhost:4222`

---

### `listener.ts` 的核心結構

```typescript
import nats from 'node-nats-streaming';

const stan = nats.connect('ticketing', '123', {
  url: 'http://localhost:4222'
});

stan.on('connect', () => {
  console.log('Listener connected to NATS');
});
```

---

### Client ID 的重要性：不能重複

當你嘗試用**相同的 Client ID** 同時啟動兩個 Listener 實例時，NATS Streaming Server 會直接拒絕第二個連線，報錯：

```
Client ID already registered
```

這是因為 Server 內部維護了一張已連線 Client 的清單，每個 Client ID 必須唯一。

**解決方案：隨機生成 Client ID**

```typescript
import { randomBytes } from 'crypto';

const clientId = randomBytes(4).toString('hex');
```

在 Kubernetes 環境中，每個 Pod 其實都有唯一的名稱，通常我們會用 Pod 名稱當作 Client ID，詳見後續章節。

---

### 如何從本地端直連 Kubernetes 內的 NATS Server？

`kubectl port-forward` 可以將叢集內 Pod 的特定連接埠映射到本機，讓本地程式能直接與叢集內的 Service 溝通。

```bash
kubectl get pods  # 先找到 NATS Pod 的名稱
kubectl port-forward <nats-pod-name> 4222:4222
```

執行後，本機的 `localhost:4222` 就會被轉送到 Kubernetes 內的 NATS Server Pod 的 4222 埠。

> **使用 `port-forward` 的時機**：這是一個**純開發用途**的技巧。對於正式環境，我們應該透過 Ingress 或 NodePort Service 暴露服務，而非依賴 port-forward。

---

### NPM Scripts 配置

```json
{
  "scripts": {
    "publish": "ts-node-dev --restart-src 0.0.0.0 --no-notify src/publisher.ts",
    "listen": "ts-node-dev --restart-src 0.0.0.0 --no-notify src/listener.ts"
  }
}
```

- **`--no-notify`** — 停用 ts-node-dev 的桌面通知功能，避免開發時干擾
- **`--restart-src 0.0.0.0`** — 允許程式從任何 IP 接收重載信號

---

## 💡 重點摘要

- **每個 NATS Client 都必須有唯一的 Client ID，否則 Server 會拒絕連線。**
- **`kubectl port-forward` 是開發期間讓本地程式直連叢集內 Service 的捷徑，但不適合正式環境。**
- **`node-nats-streaming` 使用大量 Callback 而非 `async/await`，這是正常的設計取捨。**
- **在 Kubernetes 環境中，建議用 Pod 名稱作為 Client ID，以確保唯一性。**

---

## 🔑 關鍵字

node-nats-streaming, port-forward, Client ID, Cluster ID, ts-node-dev, kubectl
