# 130 使用者屬性的型別檢查與 UserAttrs 介面

## 📝 課程概述

本單元實作 Mongoose 與 TypeScript 整合的第一步：建立 `UserAttrs` Interface 描述「建立使用者時需要傳入的參數」，並透過 `buildUser()` 工廠函式取代直接呼叫 `new User()`，讓 TypeScript 能正確檢查 Constructor 的參數型別。

---

## 核心觀念與實作解析

### 建立 Schema

```typescript
import mongoose, { Schema } from 'mongoose';

const userSchema = new Schema({
  email: { type: String, required: true },
  password: { type: String, required: true },
});

export const User = mongoose.model('User', userSchema);
```

> **注意**：`Schema` 裡的 `type: String`（大寫）是 JavaScript 的建構子，不是 TypeScript 的型別標示。兩者外表相似，意義完全不同。

### 解決問題一：建立 UserAttrs Interface

```typescript
// 描述「建立新使用者時需要提供的屬性」
interface UserAttrs {
  email: string;
  password: string;
}
```

### 解決問題一：包裝 buildUser 函式

```typescript
const buildUser = (attrs: UserAttrs) => {
  return new User(attrs);
};
```

從此**永遠不直接呼叫 `new User()`**，而是透過 `buildUser()` 建立文件。這樣 TypeScript 就能知道傳入的必須是 `{ email: string; password: string }`，多傳、傳錯型別都會被 TypeScript 攔截：

```typescript
buildUser({ emial: 'test@test' }); // 拼字錯誤，TypeScript 報錯
buildUser({ email: 'test@test', extraProp: true }); // 多餘屬性，TypeScript 報錯
buildUser({ email: 'test@test', password: 123 });   // 型別錯誤，TypeScript 報錯
```

### 為什麼不直接用 Interface 註解 Constructor？

很多人直覺會問：「為什麼不直接 `new User(attrs: UserAttrs)`？」—— 因為 Mongoose 的型別定義不支援這麼做。我們無法直接修改 Mongoose 本身，只能繞道用 wrapper function 達成相同效果。

---

## 💡 重點摘要

- `UserAttrs` Interface 描述的是「建立文件時需要傳入什麼」，而不是「文件上有哪些屬性」。
- `buildUser()` 是一層薄薄的包裝，核心價值是**強迫 TypeScript 介入 Constructor 呼叫**。
- **永遠不走捷徑**，不要直接在程式碼某處 `new User()`，否則型別檢查的防護網就失效了。

## 🔑 關鍵字

UserAttrs, buildUser, Schema, mongoose.model, TypeScript Interface
