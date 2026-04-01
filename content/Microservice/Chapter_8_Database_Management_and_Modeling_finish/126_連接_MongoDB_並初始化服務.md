# 126 連接 MongoDB 並初始化服務

## 📝 課程概述

本單元讓 auth service 實際與運行在 Kubernetes Pod 中的 MongoDB 實例建立連線。我們安裝了 Mongoose、處理了 TypeScript 型別定義問題，並將資料庫連線邏輯封裝進一個獨立的 `start()` 函式，確保連線成功後才開始接受 HTTP 流量。

## 核心觀念與實作解析

### 安裝 Mongoose 與型別定義

```bash
npm install mongoose
npm install @types/mongoose  # TypeScript 需要這個才能理解 Mongoose 的型別
```

> **為什麼需要 `@types/mongoose`？** Mongoose 本身是 JavaScript 函式庫，沒有內建的 TypeScript 型別資訊。`@types/mongoose` 提供了這些型別，讓 TypeScript 能夠對 Mongoose 的 API 進行型別檢查。

### 連線位址的組成邏輯

```typescript
await mongoose.connect('mongodb://auth-mongo-svc:27017/auth', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  useCreateIndex: true,
});
```

連線字串中 `auth-mongo-svc` 是我們剛剛建立的 Service 名稱。**在 Kubernetes 內部，各 Pod 要透過 Service 互相溝通，必須使用 Service 的名稱作為網域名稱**，而非直接使用 Pod IP——因為 Pod IP 會隨重啟改變。

| 組成部分 | 說明 |
|---|---|
| `mongodb://` | 協定前缀 |
| `auth-mongo-svc` | Service 名稱（即叢集內部 DNS 名稱）|
| `27017` | MongoDB 預設連接埠 |
| `auth` | 要連接的資料庫名稱（不存在時 Mongoose 會自動建立）|

### 為什麼需要 `start()` 函式？

```typescript
const start = async () => {
  try {
    await mongoose.connect(...);
    console.log('Connected to MongoDB');
  } catch (err) {
    console.error(err);
  }
  app.listen(3000);
};

start();
```

Node.js 的 `async/await` 語法**在舊版 Node 中不能在函式外部直接使用**。將其包進 `start()` 函式，可以確保：
1. 先等待資料庫連線完成
2. 連線成功後才執行 `app.listen()`
3. 萬一連線失敗，至少印出錯誤訊息，不至於無聲無息地失敗

### 連線失敗資料會不見嗎？

目前是會的。我們在 Kubernetes 中執行的 MongoDB Pod 沒有綁定持久化儲存（Persistent Volume）。**刪除或重啟 Pod 後，所有資料會消失**——這個問題會在後續章節處理。

## 💡 重點摘要

- Service 名稱在 Kubernetes 內部可作為 DNS 主機名，讓其他 Pod 透過 `http://service-name:port` 存取
- `mongoose.connect()` 會回傳 Promise，配合 `try/catch` 才能妥善處理連線失敗的情況
- 將 `app.listen()` 放在 `await mongoose.connect()` 之後，是確保「資料庫就緒才接受流量」的標準模式
- 目前的 MongoDB 沒有持久化儲存，Pod 重啟後資料會遺失

## 🔑 關鍵字

Mongoose, mongoose.connect, async/await, ClusterIP Service, Kubernetes DNS
