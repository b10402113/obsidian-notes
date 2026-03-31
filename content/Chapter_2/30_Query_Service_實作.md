# Query Service 實作（事件處理與資料結構）

## 📝 課程概述

本單元建立了全新的 `query` service（Port 4002），它是「非同步通訊方案」的核心成員。Query Service **不處理文章或留言的建立**，只負責監聽來自 Event Bus 的 `PostCreated` 和 `CommentCreated` 事件，逐步建構並維護一份「文章 + 留言」的本地資料副本，並提供 `GET /posts` 端點讓前端一次取回所有完整資料。

## 核心觀念與實作解析

### Query Service 的職責定位

Query Service 是「讀取視角」的代言人：
- **不擁有**文章或留言的「原始資料」
- **只持有**經過事件彙整後的「投影資料」（Projection）
- 隨時從 Event Bus 接收事件，逐步更新自己的本地狀態

這種模式稱為 **Read Model / Query Model**，與「寫入視角」的 Command Model 分離，正是 CQRS 概念的具體應用。

### 資料結構設計

```js
const posts = {};

// 結構範例：
// {
//   "post-id-1": { id: "post-id-1", title: "First Post", comments: [] },
//   "post-id-2": { id: "post-id-2", title: "Second Post", comments: [{ id: "c1", content: "Nice!" }] }
// }
```

### 事件處理邏輯

```js
// POST /events — 接收並處理事件
app.post('/events', (req, res) => {
  const { type, data } = req.body;

  if (type === 'PostCreated') {
    posts[data.id] = { id: data.id, title: data.title, comments: [] };
  }

  if (type === 'CommentCreated') {
    const post = posts[data.postId];
    post.comments.push({ id: data.id, content: data.content });
  }

  res.send({});
});
```

### `CommentCreated` 事件的 `data` 結構

```js
{
  type: 'CommentCreated',
  data: {
    id: 'comment-random-id',
    content: 'This is great!',
    postId: 'post-id-1'
  }
}
```

### GET /posts 端點

```js
app.get('/posts', (req, res) => {
  res.send(posts);
});
```

這個端點直接回傳整個 `posts` 物件，裡面已經包含了所有文章的留言——React 前端只需要一次請求，就能拿到完整的嵌套資料結構。

### 初始化時需安裝的依賴

```bash
mkdir query && cd query
npm init -y
npm install express cors nodemon
# 注意：Query Service 不需要 axios（它不發送事件，只接收）
```

## 💡 重點摘要

- **Query Service 不處理 CRUD**——它的職責是彙整（Aggregation）和投影（Projection）其他 Service 的資料。
- **CQRS 模式**：Command（寫入）與 Query（讀取）使用不同的資料模型，兩者可以各自獨立擴展。
- `posts[data.id] = { ... }` 是增量更新——即使同一個 `id` 收到多次 `PostCreated`，也不會覆蓋（因為相同 ID 不會有兩次建立）。
- `CommentCreated` 處理時，`posts[data.postId]` 可能尚未存在（若文章比留言早發出的事件尚未處理完），需注意此時 `post.comments.push()` 會報錯——但目前的測試情境下文章已存在，所以沒問題。

## 🔑 關鍵字

`Query Service`, `Read Model`, `CQRS`, `Projection`, `Event Aggregation`
