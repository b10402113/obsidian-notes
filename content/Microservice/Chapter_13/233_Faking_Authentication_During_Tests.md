# 測試中的身份驗證 — 自己組出 JWT Cookie

## 📝 課程概述

本節解決了測試環境中的身份驗證問題：Auth Service 中的 `signin()` 函式透過向 itself 發送註冊請求來取得 Cookie，但在 Tickets Service 中這樣做會引入跨服務依賴，破坏了測試的隔離性。本節的解決方案是**完全在測試環境中自己構造 JWT cookie**，繞過 HTTP 請求，直接生成有效的身份驗證憑證。

## 核心觀念與實作解析

### 為何不能呼叫 Auth Service 的註冊 API

在 Tickets Service 的測試中呼叫 `POST /api/users/signup` 會造成：

- **測試依賴另一個 Service**：如果 Auth Service 沒啟動，Tickets 的測試就會失敗。
- **測試不再是 pure 的**：測試結果會受到 Auth Service 狀態的影響。

測試應該是完全自足的（self-contained），不依賴任何外部服務。

### 手工組出 JWT Cookie 的原理

Cookie 中的 session 資料是經過 **Base64 編碼**的 JSON，字串格式如下：

```
express:sess=<base64 編碼的 JSON>
```

其中 JSON 的內容是：

```json
{ "jwt": "<actual_jwt_token>" }
```

所以我們只需要：
1. 用 `JWT.sign()` 自己建立一個 JWT token
2. 把 token 放進 session 物件
3. 用 `Buffer.from(JSON.stringify(session)).toString('base64')` 編碼
4. 組出 `express:sess=<encoded>` 字串

### 程式碼實作

```typescript
// setupTests.ts
(global as any).signin = () => {
  const payload = {
    id: new mongoose.Types.ObjectId().toHexString(),
    email: 'test@test.com',
  };
  const token = jwt.sign(payload, process.env.JWT_KEY!);
  const session = { jwt: token };
  const sessionJSON = JSON.stringify(session);
  const base64 = Buffer.from(sessionJSON).toString('base64');
  return [`express:sess=${base64}`];
};
```

**注意**：`signin()` 現在回傳的是**字串陣列**（因為 `supertest` 期望 cookie 是一個陣列），而且不再是 `async` 函式（因為我們不再發送 HTTP 請求）。

### Mongoose ObjectId 隨機生成

在測試中每次呼叫 `signin()` 都會產生一個隨機的新 ObjectId，這模擬了「不同的使用者」的概念。這為未來測試「使用者 A 無法編輯使用者 B 的票券」打下了基礎。

## 💡 重點摘要

- 測試必須完全自足，絕對不能呼叫其他 Service 的 API，否則就失去了測試隔離性。
- JWT Cookie 的底層結構是 Base64 編碼的 JSON，知道了格式就可以自己組出來。
- `Buffer.from(string).toString('base64')` 是 Node.js 內建將字串轉為 Base64 的方法。
- 每次 `signin()` 使用不同的隨機 ObjectId，模擬多個使用者的情境，對未來的擁有權測試非常重要。

## 🔑 關鍵字

JWT, Cookie, Base64, supertest, mongoose.Types.ObjectId, Test Isolation
