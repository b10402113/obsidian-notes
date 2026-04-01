# 除錯 CastError — 直接修改 node_modules 的臨時技巧

## 📝 課程概述

本節記錄了一次完整的除錯過程。測試預期收到 404，但實際收到 400，原因是 `GET /api/tickets/invalid-id` 中的 `invalid-id` 不是合法的 MongoDB ObjectId 格式，導致 Mongoose 拋出 `CastError`，而 error handler 將其視為一般錯誤回傳 400。本節介紹了一個臨時除錯技巧（在 `node_modules` 中的編譯後 JS 檔案加入 `console.error`），以及最終的解決方案：更新 Common Module 的 error handler 來記錄未預期錯誤。

## 核心觀念與實作解析

### 問題的根本原因

`CastError` 不是 `NotFoundError`（或其他任何自定義錯誤），所以它會落到 error handler 的 `else` 區塊：

```typescript
// error-handler.ts (Common)
res.status(400).send({ error: err.message });
```

即使資料庫中根本沒有這個 ID，只要格式無效就會得到 400 而非 404，這讓「ID 不存在」的測試得到錯誤的結果。

### 臨時除錯技巧：直接修改 node_modules 中的 JS 檔案

在正常的開發流程中，如果想除錯 Common Module 的 error handler，需要：

1. 修改 Common 的 TypeScript 原始碼
2. 重新 build
3. 更新 package.json 中的版本號
4. 重新 publish 到 NPM
5. 在 Tickets Service 中 `npm update`

這太繁瑣了。課程展示了一個**緊急除錯**技巧：直接打開 `node_modules/@zhx-tickets/common/build/middleware/error-handler.js`，在 `else` 區塊前插入 `console.error(err)`，就能立刻看到實際發生的錯誤是什麼。

> ⚠️ **警告**：這是非常規操作。修改 `node_modules` 的內容會在重新 `npm install` 或重建映像後被覆蓋，千萬不要將此作為正式修復手段。本技巧僅適用於「需要快速定位問題」的臨時除錯情境。

### 看到錯誤了：`CastError`

```
CastError: Cast to ObjectId failed for value "fake-id" at path "_id"
```

這告訴我們 `fake-id` 沒有通過 ObjectId 的格式驗證，Mongoose 根本沒有去查資料庫，就直接失敗了。

### 最終改善：讓 Error Handler 紀錄未預期錯誤

在 `else` 區塊中加入：

```typescript
console.error(err);
```

任何非自定義錯誤都會被記錄到標準輸出，未來除錯時就能知道「有未預期的錯誤發生了」。

### 解決測試：生成合法的 ObjectId

問題的修復不是修改 Common Module，而是在測試中使用**格式正確但不存在的 ID**：

```typescript
const id = new mongoose.Types.ObjectId().toHexString();
```

這個 ID 會成功通過 Mongoose 的格式檢查，並真正進入資料庫查詢，最後才由 `NotFoundError` 轉為正確的 404。

## 💡 重點摘要

- `CastError`（無效 ObjectId 格式）不是 `NotFoundError`，會被當作一般錯誤回傳 400。
- 在 Common Module 中加入 `console.error()` 可以幫助快速定位未預期的錯誤。
- 測試中生成有效 ObjectId 的方法：`new mongoose.Types.ObjectId().toHexString()`。
- 修改 `node_modules` 中的 JS 檔案是緊急除錯技巧，不應作為正式修復手段。

## 🔑 關鍵字

CastError, ObjectId, console.error, NotFoundError, error-handler, node_modules
