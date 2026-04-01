# 測試請求處理：Valid Input、Invalid Input 與重複註冊

## 📝 課程概述

有了第一個測試的基礎，我們現在一口氣為 Sign Up Handler 補上多個測試案例：無效 Email、無效密碼（長度不符）、缺少欄位、以及同一組 Email 不能重複註冊。同時也介紹了如何在一個測試內連續發送多個請求，並驗證資料庫的實際寫入結果。

## 核心觀念與實作解析

### 測試 Invalid Input：Email 格式錯誤

```typescript
it('returns a 400 with an invalid email', async () => {
  return request(app)
    .post('/api/users/signup')
    .send({ email: 'invalid', password: 'password' })
    .expect(400);
});
```

只要修改 Request Body 的內容與預期狀態碼，就能重複使用相同的結構。

### 測試 Invalid Input：密碼長度不符

Auth Service 的密碼驗證邏輯是長度需在 4 到 20 個字元之間。我們用一個長度為 1 的密碼來觸發錯誤：

```typescript
it('returns a 400 with an invalid password', async () => {
  return request(app)
    .post('/api/users/signup')
    .send({ email: 'test@test.com', password: 'p' })
    .expect(400);
});
```

### 測試 Missing Fields

傳送空物件會導致 `email` 與 `password` 都缺失，兩者都會觸發 Validation Error：

```typescript
it('returns a 400 with missing email and password', async () => {
  await request(app)
    .post('/api/users/signup')
    .send({})
    .expect(400);
});
```

### 單一測試內發送多個請求（async / await）

有時候需要連續執行多個操作，例如：先註冊、再以相同資料嘗試註冊，來驗證重複 Email 的錯誤處理：

```typescript
it('disallows duplicate emails', async () => {
  await request(app)
    .post('/api/users/signup')
    .send({ email: 'test@test.com', password: 'password' })
    .expect(201);

  await request(app)
    .post('/api/users/signup')
    .send({ email: 'test@test.com', password: 'password' })
    .expect(400);
});
```

> **為何這裡要 `await`？** 若第二個請求不 await，第一個請求可能尚未完成就發出，導致測試結果不穩定。在同一個測試內連續發送多個請求時，`await` 是必要的。

### 驗證 Cookie 是否被設定

除了檢查狀態碼，還可以檢查 Response Header 中的 `set-cookie` 是否存在：

```typescript
it('sets a cookie after successful signup', async () => {
  const response = await request(app)
    .post('/api/users/signup')
    .send({ email: 'test@test.com', password: 'password' })
    .expect(201);

  expect(response.get('set-cookie')).toBeDefined();
});
```

`.get('set-cookie')` 是 SuperTest Response 物件提供的方法，用來讀取 Response Header。

### 清理 Console.log

測試執行時若看到多餘的 `console.log`，通常是 Error Handler Middleware 中的除錯用的 log。建議從 Error Handler 中移除，只保留在開發初期用於手動測試的 log，避免干擾測試輸出。

## 💡 重點摘要

- **多個請求在同一測試內時，務必使用 `await`**——確保請求依序完成後再發下一個
- `.get()` 方法可以取出任意 Response Header，適合驗證 `set-cookie`
- `beforeEach` 清空 Collection 後，每個測試都從乾淨的資料庫狀態開始，測試結果才可靠
- Sign Up Handler 的錯誤處理是「任何一個 Validation 失敗就整個 fail」，所以缺少多個欄位時同樣回 400

## 🔑 關鍵字

expect(), .get(), set-cookie, async/await, duplicate email, beforeEach
