# Comments Service 實作

## 📝 課程概述

本單元實作了第二個 Express 服務——`comments` service（監聽 Port 4001）。它負責處理「對特定文章新增留言」與「取得特定文章的所有留言」兩個操作。核心資料結構採用巢狀 Map：`commentsByPostId` 以 `postId` 為 key，value 是該文章的留言陣列。

## 核心觀念與實作解析

### 資料結構設計

```js
const commentsByPostId = {};
// 結構範例：
// {
//   "post-id-1": [{ id: "c1", content: "Great post!" }],
//   "post-id-2": [{ id: "c2", content: "Nice!" }]
// }
```

以 `postId` 當作第一層 key 的原因是：**查詢效率**。當前端要取某篇文章的留言時，只需要一次查詢 `commentsByPostId[postId]`，就能拿到完整的留言陣列。

### API 設計

```js
// GET /posts/:id/comments — 取得指定文章的留言
app.get('/posts/:id/comments', (req, res) => {
  res.send(commentsByPostId[req.params.id] || []);
});

// POST /posts/:id/comments — 對指定文章新增留言
app.post('/posts/:id/comments', async (req, res) => {
  const commentId = crypto.randomBytes(4).toString('hex');
  const { content } = req.body;
  const postId = req.params.id;

  // 若該文章尚無任何留言，先初始化為空陣列
  let comments = commentsByPostId[postId];
  if (!comments) commentsByPostId[postId] = comments = [];

  comments.push({ id: commentId, content });
  res.status(201).send(comments);
});
```

### 「防禕性初始化」的設計理由

```js
let comments = commentsByPostId[postId];
if (!comments) commentsByPostId[postId] = comments = [];
```

當某篇文章是第一次被留言，`commentsByPostId[postId]` 會是 `undefined`。如果直接 `push` 會拋出錯誤。我們用「預設空陣列」的模式來安全處理這個邊界情況，這是**防禕性程式設計**的典型應用。

### 安裝與啟動

```bash
cd comments
npm init -y
npm install express body-parser axios nodemon
# 別忘了 package.json 加入 "start": "node index.js"
npm start
# 預期看到：Listening on 4001
```

### 手動測試流程

1. **建立留言**：POST `localhost:4001/posts/abc123/comments`，Body `{"content": "I am a comment"}`
2. **預期回應**：HTTP 201 + 該文章目前所有留言的陣列
3. **取得留言**：GET `localhost:4001/posts/abc123/comments`
4. **預期回應**：HTTP 200 + 留言陣列

### 微服務拆分後的「ID 繞送」問題

這裡有個值得注意的設計取捨：Comments Service 目前對任何字串當作 `postId`，**不驗證**該文章是否真的存在於 Posts Service。這是刻意為之——真實系統中，留言服務需要透過某種通訊方式向 Posts Service 確認文章有效性，這正是本章後續要解決的問題。

## 💡 重點摘要

- **`commentsByPostId`** 以 `postId` 為第一層 key，實現 O(1) 查詢效能。
- 「防禕性初始化」(`if (!comments) comments = []`) 是安全處理空值邊界情況的標準技巧。
- Comments Service 目前**不驗證** `postId` 的有效性——這預留了跨服務溝通問題的討論空間。
- 留言的回應回傳整個陣列而非單一物件，是為了方便前端直接替換狀態。

## 🔑 關鍵字

`commentsByPostId`, `Nested Map`, `Defensive Coding`, `REST API`, `微服務拆分`
