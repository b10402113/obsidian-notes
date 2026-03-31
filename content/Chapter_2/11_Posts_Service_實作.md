# Posts Service 實作

## 📝 課程概述

本單元實作了第一個 Express 服務——`posts` service。我們為它定義了兩個 REST API 端點（`GET /posts` 與 `POST /posts`），並使用 **in-memory 儲存**（JavaScript 物件）來暫存所有文章資料。這是整個微服務專案的起點。

## 核心觀念與實作解析

### 為什麼使用 In-Memory 儲存？

本專案刻意**不做資料庫設定**，目的是讓讀者聚焦在微服務之間的通訊邏輯，而非資料持久化的實作細節。缺點是：每當服務重啟，資料就會全部消失——但這對學習場景完全可接受。

### API 設計

```js
// index.js — posts service

// GET /posts → 取得所有文章
app.get('/posts', (req, res) => {
  res.send(posts);  // posts 是個 JavaScript 物件
});

// POST /posts → 建立新文章
app.post('/posts', async (req, res) => {
  const id = crypto.randomBytes(4).toString('hex');
  const { title } = req.body;
  posts[id] = { id, title };
  res.status(201).send(posts[id]);
});
```

- **`GET /posts`**：直接回傳整個 `posts` 物件（以 post ID 為 key 的 Map 結構）。
- **`POST /posts`**：從 request body 取出 `title`，用 `crypto.randomBytes(4)` 產生一個隨機 ID，將 `{ id, title }` 存入 `posts` 物件，回傳 HTTP 201（已建立）。

### Body Parser 的必要性

Express 無法自動解析 request body 中的 JSON 資料，必須透過 `body-parser` middleware：

```js
const bodyParser = require('body-parser');
app.use(bodyParser.json());
```

沒有這行，`req.body` 會是 `undefined`，POST 端點無法取得使用者傳來的 `title`。

### 安裝流程（指令回顧）

```bash
mkdir posts && cd posts
npm init -y
npm install express body-parser axios nodemon
```

### package.json 啟動腳本

```json
"scripts": {
  "start": "node index.js"
}
```

### 手動測試：用 Postman 驗證

1. 建立文章：POST `localhost:4000/posts`，Header `Content-Type: application/json`，Body `{"title": "My First Post"}`
2. 預期回應：HTTP 201 + `{ "id": "a1b2c3d4", "title": "My First Post" }`
3. 取得文章：GET `localhost:4000/posts`
4. 預期回應：整個 `posts` 物件

## 💡 重點摘要

- **`crypto.randomBytes(4)`** 產生 8 碼十六進位字串，作為文章的隨機 ID，避免碰撞。
- HTTP 201 狀態碼代表「資源已成功建立」，是 RESTful API 的標準慣例。
- **`nodemon`** 會監聽檔案變更自動重啟 server，但重啟後 in-memory 資料會消失——這是預期行為。
- Body Parser 是 Express 解析 JSON request body 的必備中介軟體。

## 🔑 關鍵字

`Express`, `Body Parser`, `In-Memory`, `REST API`, `Postman`
