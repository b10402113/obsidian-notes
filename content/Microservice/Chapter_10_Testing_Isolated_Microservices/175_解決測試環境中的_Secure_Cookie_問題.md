# 解決測試環境中的 Secure Cookie 問題

## 📝 課程概述

本節解決一個測試階段才會暴露的實務問題：當我們嘗試驗證 Sign Up 成功後有正確設定 Cookie 時，測試一直失敗。原因是 `cookie-session` Middleware 的 `secure: true` 選項只允許在 HTTPS 連線下設定 Cookie，而 SuperTest 發送的是 HTTP 請求。我們會修改 `app.ts`，讓 `secure` 選項在測試環境中自動關閉。

## 核心觀念與實作解析

### 為何 Cookie 沒有被設定？

在 `app.ts` 中，`cookie-session` 的設定如下：

```typescript
app.use(
  require('cookie-session')({
    keys: [cookieSessionKeys],
    secure: true,  // 只在 HTTPS 連線時才發送 Cookie
  })
);
```

當瀏覽器或 Postman 透過 HTTPS 訪問 Production 環境時，`secure: true` 是正確且必要的安全設定。但在測試環境中，SuperTest 發送的是**普通 HTTP 請求**（localhost），`cookie-session` 因此判定這不是安全連線，直接略過設定 Cookie——即使 Session 物件中有 JWT。

**這就是為何 `.get('set-cookie')` 一直回傳 `undefined`！**

### 解法：根據環境切換 Secure 設定

```typescript
app.use(
  require('cookie-session')({
    keys: [cookieSessionKeys],
    secure: process.env.NODE_ENV !== 'test',
  })
);
```

- 當 `NODE_ENV === 'test'` 時（ Jest 執行測試時會自動設定），`secure` 為 `false`，允許 HTTP 請求也能收到 Cookie
- 其他環境（`development`、`production`）則保持 `secure: true`

這個條件看似簡單，但背後的設計思想是：**測試環境應該盡可能模擬 Production 行為，但某些與安全傳輸層相關的設定，必須在測試中暫時放寬**。

### `NODE_ENV=test` 是 Jest 自動設定的

Jest 在啟動時會自動將 `process.env.NODE_ENV` 設為 `'test'`，所以不需要我們手動在任何地方設定這個值。

## 💡 重點摘要

- `secure: true` 是 Production 必備的安全設定，用來防止 Cookie 在 HTTP 傳輸中被截獲
- 在測試環境中，HTTP 請求無法滿足 `secure: true`，導致 `cookie-session` 不發送 Cookie
- 使用 `process.env.NODE_ENV !== 'test'` 來區分，是最乾淨且最常見的做法
- Jest 執行時自動將 `NODE_ENV` 設為 `test`，不需要手動干預

## 🔑 關鍵字

cookie-session, secure, NODE_ENV, Jest, SuperTest, set-cookie
