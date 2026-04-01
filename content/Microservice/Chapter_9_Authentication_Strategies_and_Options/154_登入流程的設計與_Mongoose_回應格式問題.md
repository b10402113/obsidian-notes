# 登入流程的設計與 Mongoose 回應格式問題

## 📝 課程概述

本單元在正式實作 Sign In 之前，先處理一個重要的周邊問題：當 Server 回傳 Mongoose Document 作為 JSON 時，`password`、`__v` 等**內部欄位不應該暴露給客戶端**。此外，MongoDB 的 `_id` 欄位命名慣例與前端預期不一致，需要統一轉換。本單元也介紹了驗證請求的 Middleware 抽取技巧。

---

## 核心觀念與實作解析

### 為什麼要過濾 Mongoose Document 的欄位？

Sign Up 成功後，Server 回傳整個 Mongoose Document，這會導致：

- **`password` 欄位暴露**：密碼的雜湊值不應出現在 API 回應中
- **`__v`（Version Key）暴露**：這是 MongoDB 內部版本控制機制，對客戶端完全無意義
- **`_id` vs `id`**：MongoDB 用 `_id`，但 MySQL/PostgreSQL 用 `id`。跨服務時，前端若要統一的錯誤訊息格式，`id` 的命名也應該一致

> **設計原則**：Document 如何在網路上傳輸，應該**由 Model 層統一控制**，而非在各 Route Handler 中重複處理。

---

### 使用 Mongoose 的 `toJSON` Transform

在 User Schema 的第二個參數（Schema Options）中設定：

```typescript
const userSchema = new mongoose.Schema({
  email: { type: String, required: true },
  password: { type: String, required: true },
}, {
  toJSON: {
    transform(doc, ret) {
      delete ret.password;           // 移除密碼
      delete ret.__v;                // 移除版本號
      ret.id = ret._id;              // _id → id
      delete ret._id;
    }
  }
});
```

**為什麼在 Schema Options 而非 Route Handler 中處理？**
- 所有回傳 User Document 的 API（Sign Up、Sign In、Current User）都自動受到保護
- 不需要四處複製貼上同一段邏輯

---

### 抽取驗證請求 Middleware

在 Sign Up / Sign In 中，檢查 Validation Errors 的邏輯高度相似：

```typescript
// ❌ 重複寫法
const errors = validationResult(req);
if (!errors.isEmpty()) {
  throw new RequestValidationError(errors.array());
}
```

將其抽取為獨立的 Middleware：

```typescript
// middleware/validate-request.ts
export const validateRequest = (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    throw new RequestValidationError(errors.array());
  }
  next();
};
```

> **注意**：這是**普通 Middleware**（3 個參數），而非 Error-Handling Middleware（4 個參數）。兩者在 Express 中由參數數量區分。

---

## 💡 重點摘要

- **在 Schema 的 `toJSON` transform 中過濾欄位，能確保所有 API 回應都符合預期格式，且不重複。**
- **MongoDB 的 `_id` 暴露在前端是壞味道，跨資料庫的微服務應該統一使用 `id`。**
- **驗證請求的錯誤處理邏輯在各 Route Handler 中完全相同，抽取成 Middleware 是正確的抽象。**

---

## 🔑 關鍵字

Mongoose Schema, toJSON Transform, Validation Middleware, password, __v, _id, Express Middleware
