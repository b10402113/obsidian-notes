# 138 密碼比對實作與 Mongoose pre('save') 中介層

## 📝 課程概述

本單元完成密碼雜湊的完整實作：建立獨立的 `Password` Service class 包含 `toHash()` 與 `compare()` 兩個 static method，並透過 Mongoose 的 `pre('save')` Hook 在每次儲存文件前自動將密碼雜湊。這樣路由處理器本身完全不需要知道密碼雜湊的細節。

---

## 核心觀念與實作解析

### Password Service 的設計

將雜湊邏輯獨立出來，好處是 `user.model.ts` 不會被雜湊實作細節汙染，讓程式碼職責分離。

```typescript
// src/services/password.ts
import { scrypt, randomBytes } from 'crypto';
import { promisify } from 'util';

const scryptAsync = promisify(scrypt);

export class Password {
  static async toHash(password: string): Promise<string> {
    const salt = randomBytes(8).toString('hex');
    const buf = (await scryptAsync(password, salt, 64)) as Buffer;
    return `${buf.toString('hex')}.${salt}`;
  }

  static async compare(storedPassword: string, suppliedPassword: string): Promise<boolean> {
    const [hashedPassword, salt] = storedPassword.split('.');
    const buf = (await scryptAsync(suppliedPassword, salt, 64)) as Buffer;
    return buf.toString('hex') === hashedPassword;
  }
}
```

> **`scrypt`** 是一個設計用來抵抗硬體暴力破解的 Hash 函式，比 MD5、SHA-1 更適合密碼儲存場景。Node.js 內建的 `crypto` 模組直接提供 `scrypt`，無需額外安裝 library。

### 為什麼要 split('.')？

`toHash()` 回傳的格式是 `{hash}.{salt}`（以 `.` 連接）。`compare()` 時需要把兩者分開，先取出原始 Salt，再用同一個 Salt 對「使用者這次輸入的密碼」重新 Hash，比對結果是否與儲存的 Hash 相同。

### Mongoose pre('save') Hook

```typescript
userSchema.pre('save', async function (done) {
  if (this.isModified('password')) {
    const hashed = await Password.toHash(this.get('password'));
    this.set('password', hashed);
  }
  done();
});
```

這段程式碼在**每次 `save()` 被呼叫前**自動執行。重點細節：

- **`function` 關鍵字（非 Arrow Function）**：Middleware 內的 `this` 綁定到正在被儲存的 Document。若用 Arrow Function，`this` 會變成外部檔案的上下文，造成錯誤。
- **`this.isModified('password')`**：確保我們只對「密碼有變動」的情況做雜湊。如果從資料庫取出使用者後只是修改 Email，就不應該再次 Hash（否則會把已經 Hash 過的密碼再 Hash 一遍）。
- **`done()` 回调**：Mongoose 的 Middleware API 預設基於 callback 而非 Promise，所以我們在異步工作完成後必須呼叫 `done()` 通知 Mongoose 繼續。

---

## 💡 重點摘要

- **`Password.toHash()`** 回傳格式固定為 `{hash}.{salt}`，整段儲存進資料庫，取出時 split('.') 即可還原。
- **pre('save') Hook** 是自動化的關鍵——路由處理器只管設定 `user.password = rawPassword`，其餘全部由 Middleware 處理。
- 使用 **`function` 關鍵字**而非 Arrow Function，是 Mongoose Middleware 的常見陷阱。

## 🔑 關鍵字

Password Service, scrypt, pre('save'), isModified, Mongoose Middleware, Salt
