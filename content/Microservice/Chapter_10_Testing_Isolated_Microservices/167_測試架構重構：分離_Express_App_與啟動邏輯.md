# 測試架構重構：分離 Express App 與啟動邏輯

## 📝 課程概述

要在測試環境中使用 SuperTest 發送假請求，我們需要對專案結構進行一次關鍵重構：將原本集中在 `index.ts` 的 Express App 創建邏輯，拆分為獨立的 `app.ts`（只負責建立與設定 App）與 `index.ts`（負責啟動服務）。這個拆分的根本原因，是避免多個 Service 的測試同時監聽同一個 Port 3000 產生衝突。

## 核心觀念與實作解析

### 為何需要重構？

使用 SuperTest 測試 Express App 時，SuperTest 會呼叫 `request(app)` 並發送 HTTP 請求。理想情況下，如果 Express App 尚未監聽任何 Port，SuperTest 會自動指派一個暫時連接埠（Ephemeral Port），讓多個測試可以在同一台機器上同時運行而不衝突。

**問題在於：** 我們原本的 `index.ts` 包含了以下啟動邏輯：

```typescript
// index.ts 中有這段 code
app.listen(3000, () => {
  console.log('Listening on port 3000!');
});
mongoose.connect MONGO_URL, ...);
```

當 `index.ts` 被 require 時，Express App 就會自動監聽 Port 3000。若同時跑 Auth Service 與 Order Service 的測試，兩邊都會嘗試監聽 3000，馬上就會噴 `EADDRINUSE` 錯誤。

### 重構策略：將 App 設定與啟動邏輯分開

我們將 `index.ts` 拆分為兩個檔案：

**`app.ts`（新檔案）**
- 只負責建立 Express App
- 連接所有 Middleware 與 Route Handlers
- **不啟動監聽**——不呼叫 `app.listen()`
- **不連接 MongoDB**——這些屬於啟動階段的職責
- **Export `app`** 供測試檔案使用

```typescript
// app.ts
import express from 'express';
import { currentUserRouter } from './routes/currentUser';
// ... 其他 imports

const app = express();
// ... 中間件設定
app.use(currentUserRouter);
// ... 其他 routers

export { app };
```

**`index.ts`（修改後）**
- 引入 `app` 來自 `app.ts`
- 負責 MongoDB 連線
- 負責啟動監聽

```typescript
// index.ts
import mongoose from 'mongoose';
import { app } from './app';

mongoose.connect(MONGO_URL, { useNewUrlParser: true, useUnifiedTopology: true });
app.listen(3000);
```

### Jest 的測試設定：為何要用 Ephemeral Port？

回到 SuperTest 的行為：當 Express App 尚未監聽連線時，SuperTest 會自動啟動它並監聽一個隨機可用的 Ephemeral Port。這正是我們需要的——不同 Service 的測試可以使用不同 Port，同時運行也互不干擾。

這也是為什麼 `app.ts` 中**絕對不能有 `app.listen()`** 的呼叫。

## 💡 重點摘要

- **拆分 `app.ts` 與 `index.ts` 的核心原因**：避免 `app.listen(3000)` 在測試環境中被觸發，導致多個 Service 測試搶 Port
- 當 Express App 未主動監聽時，SuperTest 會自動指派 Ephemeral Port，讓隔離測試成為可能
- `app.ts` 只負責設定 Middleware 與 Routes；連線與啟動邏輯留在 `index.ts`
- 這個重構不只服務測試，也讓 App 結構更清晰，未來在其他場景（如 Serverless）也更容易抽換啟動方式

## 🔑 關鍵字

SuperTest, Ephemeral Port, app.ts, index.ts, Express App Export
