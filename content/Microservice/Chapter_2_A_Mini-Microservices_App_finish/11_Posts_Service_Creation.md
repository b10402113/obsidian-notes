# Post Service 實作：Express + In-Memory Storage

## 📝 課程概述

本單元帶你從零實作第一個 Express-based Service——Post Service。我們會建立一個具備兩個 REST Endpoints 的迷你 Express 應用程式：建立文章（`POST /posts`）與取得所有文章（`GET /posts`）。資料以 in-memory 方式儲存，這個設計在這個學習階段完全可接受，因為重點在於理解 Service 的職責邊界，而非資料持久化。

---

## 核心觀念與實作解析

### 專案架構：Client + Post Service + Comment Service

我們的系統組成如下：

```
[Browser / React App] → HTTP → [Post Service :4000]
                              [Comment Service :4001]
```

- **Client**：React App，透過 HTTP 向後端 Service 發送請求
- **Post Service**：Express 應用，Listen on Port 4000
- **Comment Service**：Express 應用，Listen on Port 4001
- **資料儲存**：全部使用 in-memory（重點在理解架構，非持久化）

---

### Post Service 的 API 設計

在動手寫程式之前，先規劃好 Service 的介面（Interface）：

| Method | Path | 功能 |
|--------|------|------|
| `GET` | `/posts` | 取得所有文章 |
| `POST` | `/posts` | 建立新文章（Body: `{ title: string }`）|

---

### 實作細節：index.js

#### 1. 初始化 Express 與 Body Parser

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const crypto = require('crypto');

const app = express();
app.use(bodyParser.json());
```

#### 2. In-Memory 資料結構

```javascript
const posts = {};
```

使用物件（而非陣列）儲存文章，是因為未來可以透過 post ID 以 O(1) 的速度查詢特定文章。

#### 3. GET Route：取得所有文章

```javascript
app.get('/posts', (req, res) => {
  res.send(posts);
});
```

直接回傳整個 `posts` 物件——回傳格式為 `{ [postId]: { id, title } }`。

#### 4. POST Route：建立文章

```javascript
app.post('/posts', (req, res) => {
  const id = crypto.randomBytes(4).toString('hex');
  const { title } = req.body;

  posts[id] = { id, title };

  res.status(201).send(posts[id]);
});
```

使用 `crypto.randomBytes(4).toString('hex')` 產生一個 8 字元的隨機十六進位 ID。HTTP Status 201 表示「Created」。

---

### 使用 Postman 進行手動測試

在進入自動化測試之前，我們先用 Postman 快速驗證 Service 是否正常運作：

1. **建立文章**：POST `localhost:4000/posts`，Header 設 `Content-Type: application/json`，Body 為 `{"title": "First Post"}`
2. **預期回應**：HTTP 201 + `{ "id": "xxxx", "title": "First Post" }`
3. **取得列表**：GET `localhost:4000/posts`
4. **預期回應**：HTTP 200 + 整個 `posts` 物件

> 由於資料存在記憶體中，只要重啟 Node.js 進程（例如：儲存程式碼觸發 nodemon 自動重啟），所有文章就會消失。這在學習階段完全可接受。

---

## 💡 重點摘要

- Post Service 的設計方針是：**每個 Service 只實作一個 Resource 的 CRUD**。
- In-memory 儲存適合學習階段，但重啟後資料會消失，這是刻意為之的取捨（trade-off）。
- 用 `crypto.randomBytes(4)` 產生 8 字元的十六進位 ID，足以在小型應用中確保唯一性。
- HTTP 201 是建立資源成功時的標準回應碼，代表「已成功建立」。
- 先規劃好 API 的介面，再開始寫程式，是設計 Service 時的良好習慣。

## 關鍵字

Express.js, Body Parser, In-Memory Storage, REST API, POST /posts, GET /posts, crypto.randomBytes, HTTP 201, Postman Testing, Node.js, nodemon
