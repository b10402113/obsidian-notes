# 128 TypeScript 與 Mongoose 整合的兩大難題

## 📝 課程概述

正式建立 Mongoose User Model 之前，老師先預告了 TypeScript 與 Mongoose 合作的兩個核心痛點：Constructor 參數的型別檢查，以及 Document 上額外屬性的型別推斷。理解這些問題，才能理解後續章節「看似複雜」程式碼背後的真正原因。

---

## 核心觀念與實作解析

### 背景：MongoDB、Mongoose Model 與 Document 的術語

- **Collection（集合）**：MongoDB 中儲存一群文件的容器，相當於 RDBMS 的 Table。
- **User Model（模型類別）**：用來與整個 Collection 互動的類別——查詢、建立、修改都透過 Model。
- **User Document（文件實例）**：代表 Collection 中**單一筆**資料的類別實例。

### 問題一：Constructor 參數沒有型別檢查

```typescript
const user = new User({ email: 'test@test', password: 'whatever' });
```

當我們用 `new User(...)` 建立文件時，TypeScript **完全不知道**應該傳入哪些參數。你可以打錯字（`pasword`）、加入不存在的屬性，或是傳入錯誤型別的值，TypeScript 都不會攔截。這違背了使用 TypeScript 的初衷。

### 問題二：Document 上的額外屬性沒有被 TypeScript 知道

即使我們只傳了 `email` 和 `password` 給 Constructor，Mongoose 可能在文件上自動加上 `createdAt`、`updatedAt` 等時間戳記。這些屬性在文件上實際存在，但 TypeScript 卻一無所知。

```typescript
const user = new User({ email: 'test@test', password: 'whatever' });
console.log(user.createdAt); // TypeScript 不知道這個屬性存在
```

### 解決方向預告

這兩個問題的解決方案涉及：
1. 用 **Interface** 描述 Constructor 期望的參數形狀。
2. 用另一個 **Interface** 描述 Document 的完整形狀。
3. 在 `mongoose.model<T, U>()` 中使用 **Generics** 告知 TypeScript 這兩種形狀。

---

## 💡 重點摘要

- Mongoose 設計時並未考量 TypeScript，預設無法提供完整的型別安全保障。
- **問題一**的代價是：你能以任意形狀建立文件，完全繞過 TypeScript 的保護。
- **問題二**的代價是：文件上實際存在的屬性，TypeScript 卻認為它們不存在。

## 🔑 關鍵字

Mongoose Model, User Document, TypeScript, Interface, Generics
