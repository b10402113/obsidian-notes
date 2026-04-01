# 建立可重用的 API Client 封裝 Axios

## 📝 課程概述

本單元將上一節在 `getInitialProps` 中冗長的**環境偵測與 Header 設定邏輯抽取成一個獨立的 `buildClient` 函式**。未來任何需要從 `getInitialProps` 發送請求的地方，只需要呼叫 `buildClient(context)`，就能得到一個**已經預先設定好正確 baseURL 與 Headers 的 Axios 實例**。

## 核心觀念與實作解析

### `buildClient` 的設計

```javascript
// api/buildClient.js
import axios from 'axios';

export default async ({ req }) => {
  if (typeof window === 'undefined') {
    // 我們在 Server：需要明確的 baseURL 與 Headers
    return axios.create({
      baseURL: 'http://ingress-nginx.ingress-nginx.svc.cluster.local',
      headers: req.headers,
    });
  } else {
    // 我們在 Browser：不需要額外設定
    return axios.create({
      baseURL: '/',
    });
  }
};
```

### Axios Instance 的概念

`axios.create(config)` 回傳一個**預先配置好的 Axios 實例**，日後呼叫 `client.get()`、`client.post()` 時，會自動套用這些預設值。

### 在 `getInitialProps` 中使用

```javascript
import buildClient from '../api/buildClient';

Page.getInitialProps = async (context) => {
  const client = await buildClient(context);
  const { data } = await client.get('/api/users/currentuser');
  return data;
};
```

### `_app.js` 中 `getInitialProps` 的 Context 結構差異

> 這裡有一個 Next.js 設計上的「坑」：

- **Page Component** 的 `getInitialProps` 第一個參數：`{ req, res, pathname, ... }`
- **`_app.js`** 的 `getInitialProps` 第一個參數：`{ Component, router, ctx }`，其中 `ctx.req` 才是在 `req` 所在位置

```javascript
// 在 _app.js 的 getInitialProps 中
async getInitialProps({ Component, router, ctx }) {
  // ctx.req 才是 Node.js 請求物件
  const client = await buildClient({ req: ctx.req });
  // ...
}
```

這個 API 差異是 Next.js 框架設計上的不一致性，需要特別留意。

---

## 💡 重點摘要

- **`buildClient` 封裝了所有環境偵測邏輯，日後只需 `buildClient(context).get(url)` 即可。**
- **`axios.create()` 建立預配置實例，所有 HTTP 方法都會自動套用 baseURL 與 Headers。**
- **`_app.js` 與 Page Component 的 `getInitialProps` 接收的 Context 結構不同，`req` 在 `_app.js` 中巢狀在 `ctx.req` 底下。**

## 🔑 關鍵字

buildClient, axios.create, Axios Instance, _app.js, ctx.req, 環境偵測封裝
