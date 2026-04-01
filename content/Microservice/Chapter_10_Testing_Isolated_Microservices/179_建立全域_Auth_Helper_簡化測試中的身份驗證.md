# 建立全域 Auth Helper 簡化測試中的身份驗證

## 📝 課程概述

本節處理測試中最大的實務痛點：如何在每個測試中快速取得已驗證的 Session。SuperTest 不會自動管理 Cookie，每次 Request 都是獨立的。我們將在 `setup.ts` 中建立一個全域的 `signin()` Helper 函式，讓所有測試檔案都能一鍵取得已驗證的 Cookie，大幅簡化測試程式碼。

## 核心觀念與實作解析

### 核心問題：SuperTest 不會自動傳遞 Cookie

在瀏覽器或 Postman 中，收到 `set-cookie` 後會自動儲存並傳給後續請求。但 SuperTest 的每個 `request()` 呼叫都是獨立的，**Cookie 不會自動帶到下一個請求**。

手動解法是這樣的：

```typescript
// 每次都要這樣寫，會大量重複
const signupResponse = await request(app)
  .post('/api/users/signup')
  .send({ email: 'test@test.com', password: 'password' });

const cookie = signupResponse.get('set-cookie');

const response = await request(app)
  .get('/api/users/currentuser')
  .set('Cookie', cookie);  // 手動帶上 Cookie
```

### 建立全域 `signin()` Helper

將上述邏輯封裝到 `setup.ts`，並掛在 `global` 物件上：

```typescript
// src/test/setup.ts

declare global {
  namespace NodeJS {
    interface Global {
      signin(): Promise<string[]>;
    }
  }
}

global.signin = async () => {
  const email = 'test@test.com';
  const password = 'password';

  const response = await request(app)
    .post('/api/users/signup')
    .send({ email, password })
    .expect(201);

  return response.get('set-cookie');
};
```

**為什麼用 `global` 而不是 Export 函式？** 只是為了省去在每個測試檔案頂部重複 `import` 的麻煩。若偏好更好的模組化，也可以建立 `src/test/helpers.ts` 並 Export 後再 import。兩種做法都正確。

### 在測試中使用 `global.signin()`

```typescript
it('responds with details about the current user', async () => {
  const cookie = await global.signin();

  const response = await request(app)
    .get('/api/users/currentuser')
    .set('Cookie', cookie);

  expect(response.body.currentUser.email).toEqual('test@test.com');
});
```

所有需要認證的測試都只需要一行 `await global.signin()`，大幅降低樣板程式碼。

### `signin()` 函式的類型定義

```typescript
declare global {
  namespace NodeJS {
    interface Global {
      signin(): Promise<string[]>;
    }
  }
}
```

這段 `declare global` 告訴 TypeScript：`global.signin` 是一個回傳 `Promise<string[]>`（Cookie 陣列）的 async 函式。這與 Express 中我們用 `declare global { namespace Express { ... } }` 擴展 `Request` 物件是相同的模式。

## 💡 重點摘要

- SuperTest 每個 `request()` 都是獨立請求，Cookie 不會自動傳遞，必須手動用 `.set('Cookie', cookie)` 帶上
- `global.signin()` 將「註冊 → 取 Cookie」的流程封裝為一行，大幅提升測試可讀性
- `declare global` 是 TypeScript 用來擴展全域物件型別的語法
- `Promise<string[]>` 表示 `signin()` 回傳的是一個解析為字串陣列（Cookie 陣列）的 Promise
- 未來其他 Service（Order、Ticket）也會用到相同的 Helper，會直接複製這段程式碼過去

## 🔑 關鍵字

global.signin, set-cookie, .set(), TypeScript declare global, Promise<string[]>
