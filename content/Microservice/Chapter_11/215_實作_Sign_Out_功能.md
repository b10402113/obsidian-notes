# 實作 Sign Out 功能

## 📝 課程概述

本單元實作 Sign Out（登出）功能。我們會建立 `/auth/signout` 頁面，並使用 `useEffect` Hook 在頁面 render 時自動向 Auth Service 發送 `POST /api/users/signout` 請求，**清除伺服器端的 session**。這裡有一個重要的設計紀律：**Sign Out 請求必須從 Browser Component 發出，絕對不能寫在 `getInitialProps` 裡。**

## 核心觀念與實作解析

### 為何 Sign Out 不能寫在 `getInitialProps`？

`getInitialProps` 在 Server 端執行時，雖然可以發出 `POST /api/users/signout` 請求，但伺服器端的 HTTP 回應中的 `Set-Cookie: session=;` **不會被瀏覽器接收到**——瀏覽器才是管理 Cookie 的地方，伺服器端對此完全無感。

因此：
- **Sign Out**：必須從 Browser Component 發請求，讓瀏覽器接收到清除 Cookie 的 Header
- **抓取 currentUser（如在 Landing Page）**：可以在 Server 端（`getInitialProps`）執行

### `useEffect` 觸發一次性請求

```jsx
import { useEffect } from 'react';
import { useRequest } from '../../hooks/useRequest';
import { useRouter } from 'next/router';

export default function SignOut() {
  const router = useRouter();
  const { doRequest } = useRequest({
    url: '/api/users/signout',
    method: 'post',
    body: {},
    onSuccess: () => router.push('/'),
  });

  useEffect(() => {
    doRequest();
  }, []); // 空依賴陣列 = 只執行一次（mount 時）

  return <div>Signing you out...</div>;
}
```

- `useEffect` 搭配空依賴陣列 `[]`，確保 `doRequest()` 只在 Component **首次 mount** 時執行一次
- 成功後透過 `router.push('/')` 跳回 Landing Page

### 為何 `body` 是空物件 `{}`？

Auth Service 的 Sign Out 路由設計上接收 `POST` 請求（而非 `DELETE`），body 內容不重要，傳一個空物件作為滿足。

---

## 💡 重點摘要

- **Sign Out 的 Cookie 清除指令必須由瀏覽器接收，Sign Out 請求必須從 Browser Component 發出，**不能在 `getInitialProps` 裡做。**
- **`useEffect(() => {...}, [])` 是 React 中執行一次性副作用的標準模式。**
- **`router.push('/')` 在 `onSuccess` Callback 中觸發，確保只在使用者真的登出後才重新導向。**

## 🔑 關鍵字

useEffect, Sign Out, Cookie 清除, useRouter, onSuccess, POST /api/users/signout
