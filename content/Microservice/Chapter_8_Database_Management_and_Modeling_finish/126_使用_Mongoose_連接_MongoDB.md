# 126 使用 Mongoose 連接 MongoDB

## 📝 課程概述

本單元在 auth service 中引入 Mongoose，並建立與叢集中 MongoDB 的連線。我們會將 `index.ts` 中的 Express 啟動流程改寫為 `async/await` 語法，先等待資料庫連線成功，再開始監聽 HTTP 請求。這是確保 service 正常運作的前提條件。

---

## 核心觀念與實作解析

### 安裝 Mongoose 與類型定義

```bash
npm install mongoose
npm install @types/mongoose
```

`@types/mongoose` 讓 TypeScript 能理解 Mongoose library 內部的型別定義，否則 TypeScript 會對 `import mongoose from 'mongoose'` 報錯。

### 連線 URL 的構成

```typescript
mongoose.connect('mongodb://auth-mongo-svc:27017/auth', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  useCreateIndex: true,
});
```

為什麼 URL 是 `mongodb://auth-mongo-svc:27017/auth`？
- **`auth-mongo-svc`**：這是 ClusterIP Service 的名稱。在 Kubernetes 叢集內部，Service 名稱就等同於一個可解析的 DNS hostname。
- **`27017`**：MongoDB 預設監聽的 port。
- **`/auth`**：我們要連接的資料庫名稱。若資料庫尚不存在，Mongoose 會自動幫我們建立。

### 包裝成 async function

Node.js 版本的差異可能導致最上層 `await`（top-level await）無法使用，所以我們將啟動邏輯包裝成一個 `start()` 函式：

```typescript
const start = async () => {
  try {
    await mongoose.connect('mongodb://auth-mongo-svc:27017/auth', {
      useNewUrlParser: true,
      useUnifiedTopology: true,
      useCreateIndex: true,
    });
    console.log('Connected to MongoDB');
  } catch (err) {
    console.error(err);
  }

  app.listen(3000, () => {
    console.log('Listening on port 3000!');
  });
};

start();
```

> **`try/catch`** 區塊用來捕捉連線失敗的錯誤。若 MongoDB 尚未啟動或 URL 有誤，這段邏輯能確保 service 不會在背景靜默失敗，而是印出錯誤供開發者 Debug。

---

## 💡 重點摘要

- 叢集內 Service 之間的溝通，是透過 Service 名稱（DNS）而非 IP address。
- `mongoose.connect()` 是非同步的，必須等它完成後才能安全地處理 HTTP 請求。
- 三個 Mongoose 選項（`useNewUrlParser`、`useUnifiedTopology`、`useCreateIndex`）主要用來消除 deprecation warning，實質業務邏輯影響不大。

## 🔑 關鍵字

Mongoose, mongoose.connect, async/await, ClusterIP Service, MongoDB URL
