# 處理多個 getInitialProps 的坑

## 📝 課程概述

本單元解決 Next.js 中一個**重要的設計缺陷**：當你在 `_app.js` 定義了 `getInitialProps`，**原本各個 Page Component 的 `getInitialProps` 會被自動停用**，不再被呼叫。本節說明如何在 `_app.js` 中**手動呼叫目標 Page Component 的 `getInitialProps`**，確保兩層級的資料都能被正確抓取。

## 核心觀念與實作解析

### 問題敘述

當 `_app.js` 有 `getInitialProps` 時，Page Component（如 `index.js`）的 `getInitialProps` **不會自動執行**。

驗證方式：在 Page Component 的 `getInitialProps` 加入 `console.log`，執行後終端機沒有輸出，確認它已被跳過。

### 解決方案：在 `_app.js` 中手動呼叫

```javascript
_app.getInitialProps = async ({ Component, router, ctx }) => {
  // 抓取整個 App 層級的資料（currentUser）
  const { data: pageContext } = await client.get('/api/users/currentuser');

  // 判斷目標 Page Component 有沒有自己的 getInitialProps
  let pageProps = {};
  if (Component.getInitialProps) {
    pageProps = await Component.getInitialProps(ctx);
  }

  return {
    pageProps,
    currentUser: pageContext.currentUser,
  };
};
```

- `ctx`（即 `appContext.ctx`）是傳給 Page Component 的 Context
- 先抓 App 層級資料，再視需要呼叫 Page 的 `getInitialProps`
- **最後把所有東西一次回傳**，由 Next.js 分配給對應的 Props

### 常見陷阱：並非所有頁面都有 `getInitialProps`

像 `/auth/signup` 和 `/auth/signin` 這樣的頁面，**並未定義 `getInitialProps`**。若未做 `if (Component.getInitialProps)` 檢查就直接呼叫，會導致 `undefined is not a function` 錯誤。

---

## 💡 重點摘要

- **當 `_app.js` 有 `getInitialProps` 時，Page Component 的 `getInitialProps` 不會自動執行，必須在 `_app.js` 裡手動呼叫。**
- **`if (Component.getInitialProps)` 是防止對沒有實作 `getInitialProps` 的 Page 呼叫失敗的必要 guard。**
- **資料需要從兩層合併：`currentUser`（App 層）與 Page 特定的資料（Page 層），最後一併回傳。**

## 🔑 關鍵字

Component.getInitialProps, Page Props, 雙層 getInitialProps, _app.js, guard clause, ctx
