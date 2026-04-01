# Jest、SuperTest 與 MongoDB Memory Server 安裝與設定

## 📝 課程概述

本節進入實作階段：安裝測試所需的依賴套件，並設定 Jest 的執行環境。我們會裝上 Jest（測試執行器）、SuperTest（HTTP 請求發送）、MongoDB Memory Server（記憶體中的 MongoDB），並在 `package.json` 中註冊 `test` script。

## 核心觀念與實作解析

### 開發依賴（Dev Dependencies）

所有測試相關套件都應該作為開發依賴安裝，理由是：未來 Build Docker Image 時，這些套件**不應該被安裝到最終的 Production Image 中**。

```bash
npm install --save-dev @types/jest @types/supertest jest ts-jest mongodb-memory-server
```

### MongoDB Memory Server 的作用

**為何不用本機已安裝的 MongoDB？**

測試時若讓多個 Service 的測試 Suite 都連線到同一個 MongoDB 執行個體，會有以下問題：

- 資料狀態互相污染（Test Suite A 刪了 Test Suite B 的測試資料）
- 難以並行執行多個測試 Suite
- 測試環境不夠隔離

MongoDB Memory Server 的做法是：**每個測試 Process 都啟動一個獨立的記憶體內 MongoDB Instance**，彼此完全隔離，速度也比真正的 MongoDB 快。

> 注意：首次安裝會下載約 80MB 的 MongoDB 二進位檔，這個下載不需要每次 Rebuild Docker Image 時都重複，所以它們是 `devDependencies`。

### package.json 測試 Script 設定

```json
"scripts": {
  "test": "jest --watchAll --no-cache"
}
```

- `--watchAll`：當任何檔案變動時自動重新執行測試
- `--no-cache`：Jest 對 TypeScript 的理解有時會有快取問題，關閉快取可避免「改了 TS 檔但測試結果沒更新」的狀況

### Jest 全域設定

```json
"jest": {
  "preset": "ts-jest",
  "testEnvironment": "node",
  "setupFilesAfterEnv": ["./src/test/setup.ts"]
}
```

- `preset: "ts-jest"`：Jest 預設不理解 TypeScript，需要透過 `ts-jest` 加入支援
- `testEnvironment: "node"`：測試環境為 Node.js（非瀏覽器）
- `setupFilesAfterEnv`：Jest 啟動後、執行測試前，會先執行此檔案——這裡用來設定 MongoDB Memory Server 與測試前置作業

### setup.ts 的核心邏輯

```typescript
// src/test/setup.ts
import { MongoMemoryServer } from 'mongodb-memory-server';
import mongoose from 'mongoose';
import { app } from '../app';

let mongo: any;

beforeAll(async () => {
  mongo = new MongoMemoryServer();
  const mongoUri = await mongo.start();
  await mongoose.connect(mongoUri, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  });
});

beforeEach(async () => {
  const collections = await mongoose.connection.db!.collections();
  for (let collection of collections) {
    await collection.deleteMany({});
  }
});

afterAll(async () => {
  await mongo.stop();
  await mongoose.connection.close();
});
```

- `beforeAll`：啟動一個新的 MongoDB Memory Server，並讓 Mongoose 連線
- `beforeEach`：每個測試執行前，清空所有 Collection，確保測試之間**完全隔離**
- `afterAll`：所有測試結束後，停止 MongoDB Memory Server 並斷開連線

## 💡 重點摘要

- 測試依賴一律使用 `--save-dev`，確保 Docker Build 時不會被裝進 Production Image
- MongoDB Memory Server 為每個測試 Suite 提供獨立的記憶體資料庫，避免資料污染
- `setupFilesAfterEnv` 是集中管理測試前置作業的標準位置（資料庫啟動、變數設定等）
- `beforeEach` 清空 Collection 是確保測試可重複執行的關鍵
- `--no-cache` 旗標可解決 Jest 與 TypeScript 互動時的惱人快取問題

## 🔑 關鍵字

Jest, SuperTest, MongoDB Memory Server, ts-jest, setupFilesAfterEnv, beforeEach
