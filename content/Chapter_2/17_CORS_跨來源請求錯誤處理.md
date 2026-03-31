# CORS 跨來源請求錯誤處理

## 📝 課程概述

本單元處理了上一堂課預見的 CORS 錯誤。瀏覽器的同源政策（Same-Origin Policy）會阻止從 Port 3000（React）向 Port 4000（Posts Service）發送請求。修復方式是在**後端 Server** 安裝並設定 `cors` 這個 Express Middleware。

## 核心觀念與實作解析

### 為什麼 CORS 錯在後端修？

CORS 是一種**由伺服器控制的安全機制**。瀏覽器在發送跨來源請求前，會先發送「Preflight 預檢請求」，詢問目標伺服器：「我允許來自這個來源的請求嗎？」如果伺服器回傳的 Header 中沒有 `Access-Control-Allow-Origin` 允許名單，瀏覽器就會阻擋回應。

因此，**無論在 React 端做什麼設定都無法繞過 CORS**——必須在 Express Server 端主動加上允許 Header。

### 修復方式：安裝與使用 `cors` Middleware

```bash
# 在 posts 與 comments 兩個 Service 中執行
npm install cors
```

```js
// 在 posts/index.js 與 comments/index.js 的頂部
const cors = require('cors');
app.use(cors());
```

`cors()` 不帶任何參數時，預設允許**所有來源**的跨來源請求。這對開發環境完全可接受（Production 環境應指定明確的允許名單）。

### CORS 錯誤的典型特徵

```
Access to XMLHttpRequest at 'http://localhost:4000/posts'
from origin 'http://localhost:3000' has been blocked by CORS policy
```

這段錯誤訊息清楚說明：來自 `localhost:3000` 的請求，被 `localhost:4000` 的 CORS 政策阻擋了。

### 多 Service 的 CORS 處理原則

由於我們有多個後端 Service（Posts、Comments），**每個 Service 都必須各自設定 `cors()` middleware**。一旦某個 Service 忘記設定，瀏覽器對它的請求就會被阻擋。

> 在後期的大型專案中，架構會調整，讓前端只向一個入口（通常是 API Gateway）發送請求，CORS 問題會集中在那一處處理。

## 💡 重點摘要

- **CORS 是伺服器端的安全機制**，瀏覽器作為客戶端無權單方面放行。
- 修復 CORS 的正確位置是**後端 Express Server**，不是 React 前端。
- `cors()` 不帶參數預設允許所有來源，適合開發環境使用。
- 所有後端 Service 必須各自引入並使用 `app.use(cors())`，缺一不可。

## 🔑 關鍵字

`CORS`, `Same-Origin Policy`, `Preflight Request`, `Access-Control-Allow-Origin`, `cors Middleware`
