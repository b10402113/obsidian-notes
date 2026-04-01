# 測試 Sign Out 與 Cookie 清除行為

## 📝 課程概述

本節為 Sign Out Handler 撰寫測試。流程是：先成功 Sign Up 取得已驗證的 Session，再呼叫 Sign Out，確認伺服器正確回傳了清除 Session 的指令。我們也會學會如何透過 `console.log` 探索未知的 Response 結構，並據此撰寫斷言。

## 核心觀念與實作解析

### 測試流程：Sign Up → Sign Out

```typescript
it('clears the cookie after signing out', async () => {
  // 1. 先 Sign Up 取得已認證 Session
  await request(app)
    .post('/api/users/signup')
    .send({ email: 'test@test.com', password: 'password' })
    .expect(201);

  // 2. 呼叫 Sign Out
  const response = await request(app)
    .post('/api/users/signup')
    .send({})
    .expect(200);

  // 3. 檢查 set-cookie Header 的內容
  console.log(response.get('set-cookie'));
});
```

### 如何確認 Sign Out 的行為？

剛開始不知道 Sign Out 回傳什麼格式的 Cookie，**先用 `console.log` 觀察**：執行測試後，終端機會顯示：

```
[ 'session=; path=/; expires=Thu, 01 Jan 1970 00:00:00 GMT; httponly' ]
```

這說明 Sign Out 的機制是：
- 將 `session` 的值設為空字串 `""`
- 將 `expires` 設為過去的時間（1970），讓瀏覽器立即刪除該 Cookie

### 撰寫斷言

根據觀察結果，可以寫出精確的斷言：

```typescript
expect(response.get('set-cookie')[0]).toEqual(
  'session=; path=/; expires=Thu, 01 Jan 1970 00:00:00 GMT; httponly'
);
```

或者使用更寬鬆（也更推薦）的寫法，只要確認有設定 Cookie 即可：

```typescript
expect(response.get('set-cookie')).toBeDefined();
```

後者更優雅，因為當 Sign Out 的實作方式未來改變時（例如加密方式不同），測試不會因為字串些微差異而失敗。

## 💡 重點摘要

- `console.log` 是探索未知 Response 結構的好幫手，應該先觀察再寫斷言
- Sign Out 的清除機制是「將 Cookie 設為空值並過期」，不是刪除 Cookie 本身
- 寬鬆的斷言（如 `toBeDefined()`）比精確字串比對更具彈性，未來程式改動時較不容易出現假性失敗
- 測試 Sign Out 的前提是必須先有一個已驗證的 Session，所以流程是 `Sign Up → Sign Out`

## 🔑 關鍵字

Sign Out, set-cookie, session=;, expires, toBeDefined
