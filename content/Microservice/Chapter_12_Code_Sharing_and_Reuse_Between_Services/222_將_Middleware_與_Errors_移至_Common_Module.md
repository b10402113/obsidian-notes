# 將 Middleware 與 Errors 移至 Common Module

## 📝 課程概述

本單元實作將 Auth Service 中的 `middleware` 與 `errors` 資料夾遷移到 Common Module 的完整流程。我們會處理一個關鍵的 import 設計問題——如何讓 consumer 服務可以用最簡潔的方式引入共用程式碼，並解決遷移後 TypeScript 編譯失敗的根本原因。

---

## 核心觀念與實作解析

### 遷移策略：移動哪些資料夾？

在 Auth Service 中，**最有價值的可複用資產**是這兩個資料夾：

- `src/errors/`：包含所有自訂錯誤類別（`BadRequestError`、`NotAuthorizedError`、`NotFoundError`、`RequestValidationError`、`DatabaseConnectionError`）
- `src/middleware/`：包含 `currentUser`、`requireAuth`、`validateRequest`

將這兩個資料夾直接拖放到 `common/src/` 即完成遷移。

---

### Import 路徑的設計抉擇

遷移之後，Consumer 服務要如何引入這些程式碼？有兩種風格：

```typescript
// 風格 A：完整路徑（麻煩）
import { BadRequestError } from '@sg-tickets/common/errors/bad-request-error';

// 風格 B：直接匯出（推薦）
import { BadRequestError } from '@sg-tickets/common';
```

**風格 B 對 consumer 來說更友善**，不需要知道 Common Module 內部的資料夾結構。我們透過在 `common/src/index.ts` 中重新匯出所有模組來實現這個設計：

```typescript
// src/index.ts
export * from './errors/bad-request-error';
export * from './errors/not-authorized-error';
export * from './errors/not-found-error';
export * from './errors/request-validation-error';
export * from './errors/database-connection-error';
export * from './middleware/current-user';
export * from './middleware/require-auth';
export * from './middleware/validate-request';
```

每個 `export * from './errors/xxx'` 的意義是：從目標檔案把所有 export 內容拿進來，並立刻從 `index.ts` 再 export 出去，形成一個**轉發（re-export）**的結構。

---

### 遷移後 TypeScript 編譯失敗的根因分析

當我們執行 `tsc` 嘗試編譯 Common Module 時，TypeScript 回報了大量錯誤。**根本原因是：這些被遷移的檔案內部依賴的外部模組（`express`、`cookie-session`、`jsonwebtoken`）在 Common Module 中並不存在。**

例如，`current-user.ts` 中有這樣的 import：

```typescript
import jwt from 'jsonwebtoken';
import express from 'express';
```

這些套件只安裝在 Auth Service 的 `node_modules` 中，Common Module 並沒有它們。

**解決方法**：在 Common Module 中重新安裝所有 runtime dependencies：

```bash
npm install express cookie-session jsonwebtoken express-validator
npm install --save-dev @types/express @types/cookie-session @types/jsonwebtoken
```

> **注意**：`express-validator` 本身已內建型別定義，不需要額外安裝 `@types/express-validator`。

---

### 在 Auth Service 中更新 Import 路徑

遷移完成後，Auth Service 中原本引用這些模組的 import 語句都會斷掉。我們需要逐一修復：

**典型案例 1：`currentUser` middleware**

```typescript
// 之前
import { currentUser } from '../middleware/current-user';

// 之後
import { currentUser } from '@sg-tickets/common';
```

**典型案例 2：多個 import 合併為一個**

```typescript
// 之前
import { validateRequest } from '../../middleware/validate-request';
import { BadRequestError } from '../../errors/bad-request-error';

// 之後（合併）
import { validateRequest, BadRequestError } from '@sg-tickets/common';
```

這裡有一個額外的好處：原本散落在多個相對路徑的 import，可以**合併成一個**来自 Common Module 的 import，讓程式碼更簡潔。

**需要修復的檔案清單**：

- `src/routes/current-user.ts` → 更新 middleware import
- `src/routes/signin.ts` → 合併 `validateRequest` 與 `BadRequestError`
- `src/routes/signup.ts` → 同上
- `src/routes/signout.ts` → 通常不需要修改
- `src/app.ts` → 更新 `errorHandler` 與 `NotFoundError` 的 import

---

## 💡 重點摘要

- **Re-export 的手法讓 Common Module 的 consumer 可以用最簡潔的 `import { X } from '@org/common'` 取用所有共用元件。**
- **遷移後 TypeScript 編譯失敗不是語法問題，而是缺少 Runtime Dependencies——被遷移的程式碼依賴的 `express`、`jsonwebtoken` 等套件需要重新安裝。**
- **從 Common Module 的多個檔案 export 可以合併成一個 import，這讓 Consumer 端的程式碼更乾淨。**
- **遷移時 `src/index.ts` 的 re-export 設計是最重要的：它決定了 consumer 與我們內部實作之間的介面（API）。**

---

## 🔑 關鍵字

Re-export, `index.ts`, Common Module, Runtime Dependencies, `@types/*`, Middleware, Custom Errors
