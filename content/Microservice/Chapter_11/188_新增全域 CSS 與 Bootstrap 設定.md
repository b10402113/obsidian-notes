# 新增全域 CSS 與 Bootstrap 設定

## 📝 課程概述

本單元說明如何在 Next.js 中載入 **全域 CSS（Bootstrap）**，並介紹 Next.js 中一個特殊且關鍵的檔案：**`_app.js`**。這個檔案是 Next.js 全域初始化的進入點——無論使用者造訪哪個頁面，Next.js 都必定會載入它。

## 核心觀念與實作解析

### `_app.js` 的角色

Next.js 每次 render 任何頁面時，實際上是將該頁面的 Component 包裝在 `_app.js` 所定義的預設 App Component 內：

```
App Component（_app.js）
  └── Page Component（pages/index.js、pages/auth/signup.js 等）
```

正因為 `_app.js` 是「必定被執行」的檔案，**所有需要在每個頁面都生效的全域 CSS 必須在這裡引入**。

### `_app.js` 的結構

```javascript
import '../node_modules/bootstrap/dist/css/bootstrap.css';

export default function App({ Component, pageProps }) {
  return <Component {...pageProps} />;
}
```

- `Component`：即將 render 的目標頁面組件（如 `index.js`）
- `pageProps`：`getInitialProps` 回傳的資料（稍後章節會用到）
- 將 Bootstrap CSS 在此引入，確保每個頁面都能使用 Bootstrap 樣式

> 如果把 Bootstrap CSS 寫在 `pages/index.js` 裡，那麼當使用者瀏覽 `/auth/signup` 時，`index.js` 根本不會被載入，Bootstrap CSS 也不會生效。

### 巢狀路由與頁面群組

在 `pages/` 下建立資料夾即可建立巢狀路由：

```
pages/auth/signup.js  →  /auth/signup
```

---

## 💡 重點摘要

- **`_app.js` 是 Next.js 的全域進入點，Next.js 在 render 任何頁面時必定執行它。**
- **全域 CSS（如 Bootstrap）必須在 `_app.js` 引入，否則使用者造訪其他頁面時 CSS 會失效。**
- **`pages/` 下的資料夾結構自動對應 URL 的巢狀路徑（`pages/auth/signup.js` → `/auth/signup`）。**

## 🔑 關鍵字

_app.js, Bootstrap, 全域 CSS, pageProps, Component, 檔案系統路由
