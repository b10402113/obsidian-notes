# 134 使用者創建流程與 BadRequestError

## 📝 課程概述

本單元將 Mongoose User Model 整合進 `signup.ts` 路由處理器，完成完整的 Sign Up 流程：先檢查 Email 是否已被註冊，若無則建立使用者並存進 MongoDB。同時建立一個通用的 `BadRequestError` 類別，統一處理「使用者輸入錯誤」類型的錯誤響應。

---

## 核心觀念與實作解析

### 流程總覽

```
請求抵達 → 資料驗證 → 查詢是否已有相同 Email
  → 若有：拋出 BadRequestError
  → 若無：User.build() 建立文件 → await user.save() 寫入資料庫
  → 回傳 201 Created 與使用者資料
```

### 在路由中查詢現有使用者

```typescript
const existingUser = await User.findOne({ email });

if (existingUser) {
  throw new BadRequestError('Email in use');
}
```

`User.findOne()` 是 Mongoose Model 的查詢方法。若找不到符合的文件，回傳值為 `null`，而不是例外。

### 建立新使用者

```typescript
const user = User.build({ email, password });
await user.save();
res.status(201).send(user);
```

**重要**：只呼叫 `User.build()` 只是建立文件物件，並不會寫入資料庫。必須再呼叫 `await user.save()` 才會真正寫入。

### 建立 BadRequestError

與 `RequestValidationError` 不同，`BadRequestError` 是**通用錯誤類別**，用於任何「使用者輸入導致的失敗」情境。

```typescript
// src/errors/bad-request-error.ts
import { CustomError } from './custom-error';

export class BadRequestError extends CustomError {
  statusCode = 400;
  constructor(public message: string) {
    super(message);
    Object.setPrototypeOf(this, BadRequestError.prototype);
  }

  serializeErrors() {
    return [{ message: this.message }];
  }
}
```

---

## 💡 重點摘要

- `User.findOne()` 找不到資料時回傳 `null`，需用 `if (existingUser)` 判斷而非 try/catch。
- `User.build()` 只建立 JS 物件，**必須**再 `await user.save()` 才會寫入資料庫。
- `BadRequestError` 的 `message` 會直接出現在 API 響應中，回傳格式與其他錯誤一致。

## 🔑 關鍵字

User.findOne, User.build, user.save, BadRequestError, HTTP 201
