# 132 文件屬性定義與 TypeScript Generics 解析

## 📝 課程概述

本單元完成 Mongoose + TypeScript 整合的最後一步：建立 `UserDoc` Interface 描述 Document 的完整屬性，並將 `buildUser()` 改寫為掛在 Model 上的 **static method**，同時透過 `mongoose.model<UserDoc, UserModel>()` 的 Generics 語法，讓 TypeScript 完全理解我們的資料模型。

---

## 核心觀念與實作解析

### 建立 UserDoc Interface

```typescript
// 描述「Document 實例上實際有哪些屬性」
interface UserDoc extends mongoose.Document {
  email: string;
  password: string;
}
```

`extends mongoose.Document` 的用意是：保留 Mongoose Document 原有的所有內部屬性（如 `save()` 方法、`_id`、`__v`），再加上我們自訂的 `email` 與 `password`。

### 建立 UserModel Interface（並使用 Generics）

```typescript
interface UserModel extends mongoose.Model<UserDoc> {
  build(attrs: UserAttrs): UserDoc;
}
```

這裡告訴 TypeScript：`UserModel` 是一個 Mongoose Model，它具有一個額外的 `build` static method。

### 將 build 註冊為 Schema 的 static method

```typescript
userSchema.statics.build = function (attrs: UserAttrs) {
  return new User(attrs);
};

export const User = mongoose.model<UserDoc, UserModel>('User', userSchema);
```

`mongoose.model<UserDoc, UserModel>()` 的兩個 Generics 參數意義：
- **第一個參數（`UserDoc`）**：描述 Document 的形狀——也就是 `new User()` 或 `User.build()` 回傳值的型別。
- **第二個參數（`UserModel`）**：描述 Model 本身的形狀——也就是 `User` 這個變數本身的型別（含 static method）。

### TypeScript Generics 的直觀理解

可以把 Generics 想成「函式的隱含參數」——正常函式接收資料作為參數，而 Generics 接收**型別**作為參數：

```typescript
// model<T, U> 等價於：我們呼叫 model，並多傳了兩個「型別參數」
mongoose.model<UserDoc, UserModel>('User', userSchema);
```

---

## 💡 重點摘要

- **`UserDoc`** 描述「Document 長什麼樣」；**`UserModel`** 描述「Model（工廠函式）長什麼樣」。
- Generics 的兩個參數對應 `model` 函式的兩個**型別輸入**，不是資料。
- 以後每建立一個 Model，就複製貼上這套 Interface 模式，程式碼會高度重複，但模式穩定。

## 🔑 關鍵字

UserDoc, UserModel, Generics, mongoose.Document, userSchema.statics, mongoose.model
