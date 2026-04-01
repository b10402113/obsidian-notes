# 轉發 Headers 與 Cookie 到後端 Service

## 📝 課程概述

本單元延續上節的內容，完成**在 Server 端發送請求時轉發原始 Headers**（含 Cookie）的實作。Ingress NGINX 需要 `Host: ticketing.dev` Header 才能正確匹配路由規則；Cookie Header 則是讓 Auth Service 能識別當前使用者的關鍵。

## 核心觀念與實作解析

### `getInitialProps` 的第一個參數：Context 物件

`getInitialProps` 接收一個 Context 物件，其中包含 `req`（Node.js 的 HTTP 請求物件）：

```javascript
Page.getInitialProps = async (context) => {
  // context.req === Node.js IncomingMessage 物件
  console.log(context.req.headers);
};
```

### Ingress NGINX 為何需要 `Host` Header？

Ingress NGINX 的路由規則是基於 `host` 條件設定的（如 `ticketing.dev`）。當 Server 端直接發請求時，Axios 只發出路徑，**沒有帶上 Host Header**。Ingress NGINX 收到這樣的請求後無法匹配路由，回傳 **404**。

解決方法：手動轉發 `Host` Header：

```javascript
await axios.get(targetUrl, {
  headers: { Host: 'ticketing.dev' }
});
```

### 最完整的解法：轉發整個 Headers 物件

更通用的做法是直接轉發 `req.headers`，讓 Ingress NGINX 拿到完整的一致資訊：

```javascript
await axios.get(targetUrl, {
  headers: req.headers,
});
```

> 這麼做等於讓 Next.js Pod 成為一個**透明 Proxy**：把瀏覽器送來的所有 Header 原封不動地轉發給後端 Service。Auth Service 收到的請求幾乎與直接從瀏覽器發出的一模一樣，難以區分。

### 為何 `currentUser` 是 `null`？

成功轉發 Headers 後，如果仍看到 `null`，原因很簡單：**伺服器端發請求時壓根就沒有 Cookie**。

當使用者在瀏覽器輸入 URL 或做 Hard Refresh 時，瀏覽器確實發出了 Cookie，但 Next.js 的 SSR 流程中，需要**明確將這些 Headers 轉發**到後續請求上。

轉發後，Auth Service 就能看到 Cookie，回傳正確的 `currentUser` 資料。

---

## 💡 重點摘要

- **Ingress NGINX 的路由匹配依賴 `Host` Header，Server 端發請求若不指定，會得到 404。**
- **轉發 `req.headers` 是最完整且最易維護的做法，能同時解決 Host、Cookie 等所有 Header 的問題。**
- **`currentUser` 是 `null` 代表 Cookie 未成功轉發，確認 Headers 轉發設定後即可修復。**

## 🔑 關鍵字

Host Header, req.headers, Ingress NGINX 路由, Cookie 轉發, 透明 Proxy, IncomingMessage
