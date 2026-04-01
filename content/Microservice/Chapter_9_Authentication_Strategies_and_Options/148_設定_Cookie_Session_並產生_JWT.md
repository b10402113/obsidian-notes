# 設定 Cookie Session 並產生 JWT

## 📝 課程概述

本單元終於從理論進入實作，開始實作驗證流程。我們將安裝 `cookie-session` 與 `jsonwebtoken` 兩個 library，並在 Express 中設定 Middleware，讓系統能夠在登入/註冊時產生 JWT 並存入 Cookie。理解這些設定背後的「為什麼」同樣重要。

---

## 核心觀念與實作解析

### 安裝依賴

```bash
npm install cookie-session
npm install @types/cookie-session  # TypeScript 類型定義

npm install jsonwebtoken
npm install @types/jsonwebtoken   # TypeScript 類型定義
```

> **注意**：`cookie-session` 和 `jsonwebtoken` 都**沒有內建 TypeScript 支援**，必須額外安裝類型定義檔。

---

### 設定 cookie-session Middleware

```typescript
import cookieSession from 'cookie-session';

app.use(cookieSession({
  signed: false,       // 不加密（JWT 本身已防篡改）
  secure: true,        // 僅在使用 HTTPS 時才傳送 Cookie
}));
```

#### 為什麼 `signed: false`（不加密）？

Cookie 的內容加密與否是一個常見的安全爭議。我們**不安裝加密**的原因：

- JWT 本身**已有數位簽章**，一旦被篡改，驗證時馬上會失敗
- 若加密 Cookie，在不同語言（Node.js vs Ruby on Rails）的 Service 要解密，**增加了跨語言實作的複雜度**
- Cookie 加密只為防止「使用者讀取內容」，但**我們根本不在乎使用者能否閱讀**（JWT 的 Payload 本來就是設計成誰都能看的）

#### 為什麼 `secure: true`？

確保 Cookie **僅在 HTTPS 連線下才會被傳送**，防止 HTTP 明文傳輸時被截取。

#### 為什麼需要 `app.set('trust proxy', true)`？

當 Express 位於 Ingress Nginx 反向代理之後，Express 預設不信任代理轉發過來的 HTTPS 流量。`trust proxy` 告訴 Express：「我是被代理的，請信任這個連線是安全的。」

---

### 產生並儲存 JWT

```typescript
import jwt from 'jsonwebtoken';

const userJWT = jwt.sign(
  { id: existingUser.id, email: existingUser.email },  // Payload
  process.env.JWT_KEY!                                  // 簽章金鑰
);

// 存入 Cookie
req.session = { jwt: userJWT };
```

> **為什麼不直接傳回 JWT 而要用 Cookie？** 因為 Cookie 會讓瀏覽器在未來所有對 `ticketing.dev` 的請求中**自動攜帶**這個 JWT，包括 SSR 的第一個請求。

---

### 測試時的坑：HTTPS 問題

當使用 Postman 對 `ticketing.dev`（而非 `https://ticketing.dev`）發出請求時，Cookie 不會被設定，因為 `secure: true` 要求 HTTPS。解決方法：
1. 將 URL 改為 `https://ticketing.dev`
2. 在 Postman 設定中**關閉 SSL Certificate Verification**（因為 Ingress Nginx 使用的是自我簽署憑證）

---

## 💡 重點摘要

- **`signed: false` 是安全的，因為 JWT 的簽章機制已足以防篡改，不需要再對 Cookie 加密。**
- **`secure: true` + `trust proxy` 這兩個設定缺一不可，否則在部署環境中 Cookie 無法正常運作。**
- **使用 `req.session = { jwt: ... }` 存入 JWT 後，`cookie-session` 會自動處理序列化並發送 Set-Cookie header。**

---

## 🔑 關鍵字

cookie-session, jsonwebtoken, JWT, Set-Cookie, HTTPS, Trust Proxy, Middleware
