# 實作 Sign In 處理邏輯

## 📝 課程概述

本單元延續上一個單元的流程設計，實作完整的 Sign In 程式碼。我們將完成密碼比對、產生 JWT、存入 Cookie 的全部實作細節，並透過 Postman 測試各種情境（正確憑證、錯誤密碼、不存在的 email）。

---

## 核心觀念與實作解析

### 完整 Sign In 實作

```typescript
import jwt from 'jsonwebtoken';
import { Password } from '../services/password';
import { BadRequestError } from '../errors/bad-request-error';

router.post('/api/users/signin', async (req, res) => {
  // 1. 查詢使用者
  const existingUser = await User.findOne({ email: req.body.email });

  if (!existingUser) {
    throw new BadRequestError('Invalid credentials');
  }

  // 2. 比對密碼
  const passwordsMatch = await Password.compare(
    existingUser.password,
    req.body.password
  );

  if (!passwordsMatch) {
    throw new BadRequestError('Invalid credentials');
  }

  // 3. 產生 JWT 並存入 Cookie
  const userJWT = jwt.sign(
    { id: existingUser.id, email: existingUser.email },
    process.env.JWT_KEY!
  );

  req.session = { jwt: userJWT };

  // 4. 回傳使用者資訊
  res.send({ user: existingUser });
});
```

### Postman 測試案例

| 情境 | 預期結果 |
|------|---------|
| 正確 email + 正確密碼 | `200`，Cookie 中帶有 JWT |
| 正確 email + 錯誤密碼 | `400 BadRequestError: Invalid credentials` |
| 不存在的 email | `400 BadRequestError: Invalid credentials`（不透露具體原因）|
| 空密碼 | `400`（由 `express-validator` 的 `isNotEmpty()` 攔截）|

---

## 💡 重點摘要

- **Sign In 的查詢使用者與密碼比對邏輯都應使用統一的 `BadRequestError`，避免透過錯誤訊息區分攻擊者的猜測方向。**
- **JWT 存入 Cookie 的方式與 Sign Up 完全相同，這段程式碼在兩個 Handler 中高度重複——未來可以抽取為共用函數。**

---

## 🔑 關鍵字

Sign In, jwt.sign, BadRequestError, Password.compare, Session, Postman
