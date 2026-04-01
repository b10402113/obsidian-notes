# Server-Side Rendering 運作機制解析

## 📝 課程概述

本單元深入剖析 **Next.js 的 Server-Side Rendering 流程**。我們會看到 `getInitialProps` 這個靜態方法在 SSR 階段被呼叫，得以在伺服器端向 Auth Service 發送請求、取得「當前登入狀態」，並將結果作為 **Props** 注入 Component，確保 Landing Page 在第一次載入時就能顯示正確的登入狀態。

## 核心觀念與實作解析

### Next.js render 時的完整流程

```
瀏覽器請求抵達 Next.js Pod
  → Next.js 根據 URL 決定要 render 哪個 Page Component
  → 呼叫該 Component 的 getInitialProps（伺服器端執行一次）
  → 將 getInitialProps 回傳的資料注入 Component 作為 Props
  → Component render 一次，產生完整 HTML
  → 回傳給瀏覽器（此時使用者已能看到內容）
```

> **關鍵點**：在 SSR 階段，Component 的 `getInitialProps` 是在**伺服器**上執行的，**只能同步等待資料回來才能繼續**，Component 的 render 也只會發生一次。

### `getInitialProps` 的基本用法

```javascript
LandingPage.getInitialProps = async () => {
  console.log('I am on the server');
  return { color: 'red' };
};

function LandingPage({ color }) {
  console.log('I am in the component', color);
  return <h1>Landing Page - Color: {color}</h1>;
}
```

在終端機（伺服器端）會看到 `I am on the server`；在瀏覽器 Console（客戶端）會看到 `I am in the component red`。

### 為何在 SSR 階段不能直接用 `useRequest` Hook？

`useRequest` 是 React Hook，只能在 Component 內部執行。Component 在 SSR 時**只會 render 一次**，沒有機會等待非同步請求的結果再更新 state。**所有資料抓取都必須在 `getInitialProps` 裡完成**，因為 Next.js 會等 `getInitialProps` 的 Promise resolve 後才送出 HTML。

### 使用 Axios 直接發請求

```javascript
import axios from 'axios';

LandingPage.getInitialProps = async () => {
  const response = await axios.get('/api/users/currentuser');
  return response.data;  // { currentUser: {...} | null }
};
```

---

## 💡 重點摘要

- **`getInitialProps` 在 SSR 階段由 Next.js 自動呼叫，可用來預先抓取資料並注入 Component Props。**
- **SSR 階段 Component 只 render 一次，無法使用 Hooks 進行非同步資料更新；所有資料請求必須寫在 `getInitialProps` 裡。**
- **`getInitialProps` 回傳的資料會作為 Props 傳入 Component，Component 無需額外 fetch，直接取用即可。**

## 🔑 關鍵字

getInitialProps, SSR, Next.js, Component Props, axios.get, 非同步資料抓取
