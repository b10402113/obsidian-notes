# 請求驗證 — express-validator 與 validateRequest Middleware

## 📝 課程概述

本節實作了 Create Ticket 路由的輸入驗證邏輯。我們使用 `express-validator` 的 `body()` 函式來定義 `title` 與 `price` 的驗證規則，並透過 `validateRequest` middleware 來檢查是否有驗證錯誤。本節的重點在於理解驗證框架的工作方式，以及為何要將驗證失敗視為一個可預期的業務邏輯錯誤（400 Bad Request）而非伺服器崩潰（500）。

## 核心觀念與實作解析

### 驗證規則的設定

```typescript
import { body } from 'express-validator';
import { validateRequest } from '@zhx-tickets/common';

router.post(
  '/api/tickets',
  requireAuth,   // 先確認身份，否則不需要浪費時間驗證輸入
  [
    body('title')
      .notEmpty()
      .withMessage('Title is required'),
    body('price')
      .isFloat({ gt: 0 })
      .withMessage('Price must be greater than 0'),
  ],
  validateRequest,
  async (req, res) => { /* ... */ }
);
```

### Middleware 的順序

**先 `requireAuth`，再驗證輸入**。這個順序非常重要：

1. 如果使用者未登入，`requireAuth` 會直接拋出 `NotAuthorizedError`（401），不會浪費資源去驗證 body 內容。
2. 如果先驗證輸入再檢查身份，使用者未登入時仍然會得到一個 `RequestValidationError`（400）而不是 `NotAuthorizedError`（401），回傳的錯誤訊息會誤導使用者。

### 驗證器不會主動拋錯

`body('title').notEmpty()` 這類驗證器呼叫後**不會直接拋出錯誤**，而是把錯誤資訊附加到 `req` 物件上。必須搭配 `validateRequest` middleware 來實際檢查並拋出 `RequestValidationError`。

### 為何 price 用 `isFloat({ gt: 0 })`

票價不能為負數或零，而且必須是帶小數點的數值（`isFloat` vs `isInt`）。在金融應用中，使用 `isFloat` 而非 `isInt` 是為了支援「$9.99」這類有分幣的定價。

## 💡 重點摘要

- Middleware 的執行順序會影響錯誤類型與使用者體驗：先認證再驗證是最佳實踐。
- `express-validator` 的驗證器只是往 `req` 附加錯誤資訊，真正拋錯的是 `validateRequest` middleware。
- `isFloat({ gt: 0 })` 同時檢查了型別（小數）與範圍（大於零），滿足了票務系統的基本需求。
- 驗證失敗應該回傳 400，而非 500——這是一個使用者輸入問題，不是伺服器的問題。

## 🔑 關鍵字

express-validator, validateRequest, Middleware, body, isFloat, requireAuth
