# 登入（Sign In）的完整流程設計

## 📝 課程概述

本單元實作 Sign In Route Handler 的完整流程：接收 email + password → 查詢資料庫比對使用者 → 密碼比對（bcrypt）→ 產生 JWT 並寫入 Cookie。過程中也示範了如何自訂 `express-validator` 的錯誤訊息，以及「不透露失敗原因」的資安考量。

---

## 核心觀念與實作解析

### Sign In 的五步流程

```
1. [驗證請求] email 格式正確、password 非空
         ↓
2. [查詢使用者] User.findOne({ email })
         ↓
3. [比對密碼] Password.compare(storedHash, suppliedPassword)
         ↓
4. [失敗時不回傳具體原因] BadRequestError("Invalid credentials")
         ↓
5. [產生 JWT + 存入 Cookie] jwt.sign() → req.session = { jwt }
```

### 為什麼失敗時不透露「使用者不存在」？

當使用者輸入不存在的 email 時，許多開發者會回傳 `401 Unauthorized` 並說「此 email 未註冊」。但這等於告訴攻擊者：「這個 email 已經是我們的會員。」

**正確做法**：不區分「email 不存在」和「密碼錯誤」，統一回傳 `400 Bad Request` 並顯示「登入失敗」（或 `Invalid credentials`）。

### 使用 bcrypt 比對密碼

```typescript
import { Password } from '../services/password';

// 在 try 區塊中
const passwordsMatch = await Password.compare(
  existingUser.password,   // 已雜湊儲存的密碼
  req.body.password        // 使用者輸入的明文密碼
);

if (!passwordsMatch) {
  throw new BadRequestError('Invalid credentials');
}
```

`Password.compare` 內部會呼叫 `bcrypt.compare`，這是一個**時間正比於密碼長度**的比較函數，能防止時序攻擊（Timing Attack）。

---

## 💡 重點摘要

- **Sign In 的錯誤訊息應統一為「登入失敗」，而非區分「帳號不存在」或「密碼錯誤」，防止資訊洩漏。**
- **bcrypt.compare 是非同步的，必須使用 `await`，忘記 `await` 會導致密碼比對永遠失敗。**
- **bcrypt.compare 的時序恆定特性，能防止攻擊者透過回應時間差異推斷哪些 email 已註冊。**

---

## 🔑 關鍵字

Sign In, bcrypt, Password.compare, BadRequestError, Email Lookup, bcrypt.compare
