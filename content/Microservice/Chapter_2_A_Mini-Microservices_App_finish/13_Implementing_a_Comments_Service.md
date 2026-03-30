# Comment Service 實作：嵌套資源的資料結構設計

## 📝 課程概述

本單元實作 Comment Service，這是整個專案中第一個具有「資料依賴」特性的 Service。Comment 必須與某篇 Post 綁定，這使得它的資料結構設計與 API 設計都比 Post Service 更複雜。我們會介紹 `commentsByPostId` 這個巢狀物件結構，並透過 Postman 完成手動測試。

---

## 核心觀念與實作解析

### Comment Service 的 API 設計

| Method | Path | 功能 |
|--------|------|------|
| `GET` | `/posts/:id/comments` | 取得某篇文章的所有留言 |
| `POST` | `/posts/:id/comments` | 對某篇文章新增留言（Body: `{ content: string }`）|

注意 URL 的設計：我們把 Post ID 直接放在路徑中，這是一種常見的 RESTful 設計，稱為 **nested resource（巢狀資源）**。

---

### 資料結構：`commentsByPostId`

這裡的設計決策非常關鍵。留言資料不能只存在一個大陣列裡——因為當我们需要「取得某篇文章的留言」時，如果所有留言都混在一起，就必須遍歷整個資料集。更好的方式是：**用 Post ID 作為最外層的 key**。

```javascript
const commentsByPostId = {};
// 結構：
// {
//   "post-id-1": [
//     { id: "comment-id-1", content: "留言內容" },
//     { id: "comment-id-2", content: "另一則留言" }
//   ],
//   "post-id-2": [ ... ]
// }
```

這樣查詢某篇文章的所有留言，時間複雜度為 O(1)——只要 `commentsByPostId[postId]` 就能取得該文章的所有留言。

---

### POST Route：建立留言

```javascript
app.post('/posts/:id/comments', (req, res) => {
  const commentId = crypto.randomBytes(4).toString('hex');
  const { content } = req.body;

  let comments = commentsByPostId[req.params.id];
  if (!comments) comments = [];

  comments.push({ id: commentId, content });
  commentsByPostId[req.params.id] = comments;

  res.status(201).send(comments);
});
```

流程說明：
1. 用 `crypto.randomBytes` 產生留言 ID
2. 從 request body 取出 `content`
3. 根據 Post ID 取得（或初始化）留言陣列
4. 將新留言加入陣列後，更新 `commentsByPostId`
5. 回傳**整個陣列**（而非只回傳新建立的留言），方便 client 直接替換本地狀態

---

### GET Route：取得留言

```javascript
app.get('/posts/:id/comments', (req, res) => {
  const comments = commentsByPostId[req.params.id] || [];
  res.send(comments);
});
```

若該 Post 尚無任何留言，回傳空陣列而非 `undefined`，確保 client 端收到的是一個確定的陣列格式。

---

### 為什麼回傳整個陣列而不是只回傳新留言？

在這個情境下，client（React App）在收到成功回應後，可以直接用回傳的整個陣列**替換（replace）**本地的 comments state。這種模式稱為 **full state replacement**，好處是 client 不需要自己手動 `push` 新留言到本地 state，只需要：

```javascript
setComments(response.data);
```

即可完成更新，省去了手動 merge 的邏輯。

---

## 💡 重點摘要

- Comment Service 的 URL 採用 nested resource 設計（`/posts/:id/comments`），符合 RESTful 風格。
- `commentsByPostId` 的資料結構讓查詢特定文章的留言時間複雜度為 O(1)。
- Post ID 目前沒有經過驗證——Comment Service 目前會接受任何字串作為 Post ID，這是後續要透過事件同步機制來解決的問題。
- 回傳完整陣列而非單筆資料，是為了讓 client 更容易做 full state replacement。
- 這個 Service 完全沒有連接到 Post Service，**目前是完全獨立的**——這正是 microservices 的起點。

## 關鍵字

Nested Resource, commentsByPostId, Full State Replacement, In-Memory Storage, O(1) Lookup, POST /posts/:id/comments, GET /posts/:id/comments, RESTful API Design
