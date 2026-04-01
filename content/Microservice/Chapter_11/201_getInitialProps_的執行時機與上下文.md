# getInitialProps 的執行時機與上下文

## 📝 課程概述

本單元詳細說明 **`getInitialProps` 的觸發時機**（Server 端 vs Browser 端），以及如何在函式內**判斷當前是運行在 Node.js 環境還是瀏覽器環境**，從而決定請求的 `baseURL` 該如何設定。這是實作一個通用 API Client 的關鍵基礎。

## 核心觀念與實作解析

### `getInitialProps` 觸發時機表

| 觸發方式 | `getInitialProps` 執行位置 |
|----------|---------------------------|
| 網址列直接輸入後按 Enter | **Server** |
| 點擊來自外部網站（如 Reddit）的連結進入 | **Server** |
| 頁面 Hard Refresh（F5） | **Server** |
| 在 App 內部點擊連結跳轉（Client Navigation） | **Browser（Client）** |

### 為何要區分 Server 端與 Browser 端？

Browser 端的 Axios 請求不需要設定 `baseURL`——瀏覽器會自動補上當前網域。但 Server 端的 Axios 請求**需要明確指定**要送往哪裡（`http://ingress-nginx.ingress-nginx.svc.cluster.local`）。

### 如何判斷當前環境？

```javascript
if (typeof window === 'undefined') {
  // 我們在 Server（Node.js）環境
} else {
  // 我們在 Browser 環境
}
```

`window` 物件**只存在於瀏覽器**。Node.js 環境中 `window` 是 `undefined`。這個檢查是 Next.js 官方推薦的環境偵測方式。

### 環境偵測與對應的請求設定

```javascript
async getInitialProps() {
  if (typeof window === 'undefined') {
    // Server：明確指定 baseURL
    const response = await axios.get(
      'http://ingress-nginx.ingress-nginx.svc.cluster.local/api/users/currentuser',
      { headers: req.headers }  // 轉發所有原始請求 Header（含 Cookie）
    );
  } else {
    // Browser：讓瀏覽器自動處理網域
    const response = await axios.get('/api/users/currentuser');
  }
  return response.data;
}
```

---

## 💡 重點摘要

- **`getInitialProps` 並非只在 Server 端執行——在 App 內部導航時會在 Browser 端執行。**
- **`typeof window === 'undefined'` 是 Node.js 環境偵測的標準手法。**
- **Server 端發請求需要明確指定完整 URL（`ingress-nginx.<namespace>.svc.cluster.local`）與 Header；Browser 端只需要相對路徑。**
- **原始請求的所有 Headers（含 Cookie）必須轉發到後續請求，否則 Auth Service 收到的會是匿名請求。**

## 🔑 關鍵字

getInitialProps, 環境偵測, window, 跨 Namespace, Header 轉發, Server vs Browser
