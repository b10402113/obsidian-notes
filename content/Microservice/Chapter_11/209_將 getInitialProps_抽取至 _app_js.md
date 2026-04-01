# 將 getInitialProps 抽取至 _app.js

## 📝 課程概述

本單元將「取得當前登入狀態」的邏輯從 `pages/index.js` 移動到 `_app.js` 的 `getInitialProps`。這麼做的好處是：**所有頁面都能共享同一份 currentUser 資料**，而不需要在每個頁面各自重複實作。Header 元件因此也能正確顯示登入狀態。

## 核心觀念與實作解析

### 為何要將邏輯抽取到 `_app.js`？

目前 Landing Page 已經透過 `getInitialProps` 取得 `currentUser`，但 Header（頂部導航列）也需要知道 `currentUser`。

若每個頁面各自維護 `getInitialProps`，會造成：
1. **多個重複的 API 請求**（每個頁面都 call `/api/users/currentuser`）
2. **Header 元件無法取得 `currentUser` 資料**（Header 不是 Page Component，沒有 `getInitialProps`）

將 `getInitialProps` 拉到 `_app.js` 執行，可確保：
- `currentUser` 在**每個頁面 render 前**就已經取得
- 透過 Props 向下傳遞給 Header 與各個 Page Component

### `_app.js` 中的 `getInitialProps` Context 差異

在 `_app.js` 裡，`getInitialProps` 的第一個參數結構與 Page Component 不同：

```javascript
_app.getInitialProps = async ({ Component, router, ctx }) => {
  // 在 _app.js 中，req 在 ctx.req 底下！
  const client = await buildClient({ req: ctx.req });
  // ...
};
```

> 這是 Next.js 框架設計上的不一致性，`_app.js` 的 Context 包了一層 `ctx`，而 Page Component 的 Context 是直接展開的。

### Props 的傳遞方式

`_app.js` 的 `getInitialProps` 回傳的資料會成為 `_app` Component 的 Props。要把這些 Props 傳給目標 Page Component，需要將它們放在 `pageProps` 中：

```javascript
// _app.js
export default function App({ Component, pageProps, currentUser }) {
  return (
    <>
      <Header currentUser={currentUser} />
      <Component {...pageProps} />
    </>
  );
}
```

---

## 💡 重點摘要

- **將 `getInitialProps` 拉到 `_app.js` 能讓所有頁面共享同一份 `currentUser`，避免重複請求。**
- **`_app.js` 的 `getInitialProps` Context 結構與 Page Component 不同，`req` 藏在 `ctx.req` 底下。**
- **`pageProps` 是 Next.js 內建的 Props 傳遞橋樑，用來將 `_app.js` 的資料轉給具體的 Page Component。**

## 🔑 關鍵字

_app.js, getInitialProps, pageProps, currentUser, ctx.req, Props 傳遞
