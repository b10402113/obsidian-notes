# Current User Route 實作與 JWT 驗證

## 📝 課程概述

本單元實作 `GET /api/users/currentuser` 路由，用於讓前端應用程式查詢「目前是否有使用者登入」。我們將看到如何從 Cookie 中取出 JWT、驗證其簽章、解析 Payload，並回傳給客戶端——這是理解整個驗證流程的關鍵環節。

---

## 核心觀念與實作解析

### Current User 的工作流程

```
瀏覽器發出 GET /api/users/currentuser（自動帶上 Cookie）
         ↓
Express 收到請求，cookie-session Middleware 解析
         ↓
req.session.jwt  ← JWT 字串（若有）
         ↓
jwt.verify(jwt, JWT_KEY)  ← 驗證簽章、解碼 Payload
         ↓
回傳 { currentUser: payload } 或 { currentUser: null }
```

### 為什麼需要 Try-Catch？

`jwt.verify()` 在以下兩種情況會**拋出例外**：
1. **JWT 已被篡改**（攻擊者修改了 Payload）
2. **JWT 已過期**

這兩種情況對客戶端而言，都代表「未登入」，所以用 Try-Catch 包住即可：

```typescript
try {
  const payload = jwt.verify(req.session!.jwt!, process.env.JWT_KEY!);
  res.send({ currentUser: payload });
} catch (err) {
  res.send({ currentUser: null });
}
```

### Optional Chaining 處理 session 可能為 null

`req.session` 的型別在 TypeScript 中可能是 `null | Session`，因此存取 `.jwt` 前必須先確認 session 存在：

```typescript
if (!req.session || !req.session.jwt) {
  return res.send({ currentUser: null });
}
```

或使用 Optional Chaining：
```typescript
if (!req.session?.jwt) {
  return res.send({ currentUser: null });
}
```

> `?.` 運算子等同於「先檢查左邊是否為 null/undefined，若不是才繼續存取右邊屬性」。

---

## 💡 重點摘要

- **`jwt.verify()` 拋出錯誤並非「程式錯誤」，而是「驗證失敗」的正常流程，必須用 Try-Catch 處理。**
- **攻擊者即使能看到 JWT Payload（base64 解碼即可），也無法在不知道 Signing Key 的情況下產生有效簽章——這是 JWT 安全的核心。**
- **使用 Optional Chaining (`?.`) 是處理可能為 null 的物件時的安全寫法。**

---

## 🔑 關鍵字

jwt.verify, Current User, Payload, Optional Chaining, Session, Try-Catch
