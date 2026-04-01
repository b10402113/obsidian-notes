# 登出（Sign Out）的實作

## 📝 課程概述

本單元實作 Sign Out 路由。令人驚訝的是，Sign Out 在 Cookie-based 驗證中的實作**極度簡單**：只需將 Session 設為 `null`，讓瀏覽器自動清除 Cookie。理解背後的設計邏輯，比單純複製程式碼更有價值。

---

## 核心觀念與實作解析

### Sign Out 的原理

**HTTP 是無狀態的**，伺服器無法主動「刪除」客戶端的 Cookie。因此「登出」的實際意義是：

> **發給瀏覽器一個 `Set-Cookie: session=null`（或過期）的 header，讓瀏覽器覆寫舊的 Cookie。**

當未來瀏覽器再發送請求時，自動帶上的將是**空的或過期的 Cookie**，相當於「未登入」。

### 實作方式

```typescript
router.post('/api/users/signout', (req, res) => {
  req.session = null;  // cookie-session 會自動發送「清除 session」的 Set-Cookie
  res.send({});
});
```

`cookie-session` 監聽到 `req.session = null` 時，會自動生成一個 `Set-Cookie` header，將 Cookie 設定為**立即過期**或**清空內容**，瀏覽器收到後會移除本地儲存的 Cookie。

### 為什麼比想像中簡單？

在傳統的 Server-Side Session 方案（如 `express-session` + Redis）中，Sign Out 需要：
1. 刪除伺服器端 Redis 中的 Session 資料
2. 發送清除 Cookie 的指令

但在我們的架構中：
- JWT 是**無狀態的**，沒有伺服器端狀態需要清除
- 只需要告訴瀏覽器「忘記這個 Cookie」

---

## 💡 重點摘要

- **Sign Out 的核心是發送一個清除 Cookie 的指令，由 `cookie-session` 自動處理，不需要手動管理 Session 資料庫。**
- **JWT 作為 Stateless Token，沒有「在伺服器端撤銷」的需求——這是它比傳統 Session ID 更簡潔的原因，也是它的安全限制。**

---

## 🔑 關鍵字

Sign Out, Session, req.session = null, Cookie, cookie-session, Stateless
