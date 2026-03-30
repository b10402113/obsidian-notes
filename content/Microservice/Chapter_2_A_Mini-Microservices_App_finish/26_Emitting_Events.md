# 實作事件發射：Post Service 與 Comment Service 整合 Event Bus

## 📝 課程概述

本單元將 Post Service 與 Comment Service 分別與 Event Bus 整合，讓它們在完成本地資料庫寫入的同時，自動向 Event Bus 發射事件。這是事件驅動架構的關鍵一步——從現在起，每個 Service 的狀態變更不再只是內部行為，而是會**通知整個系統中的其他成員**。

---

## 核心觀念與實作解析

### 為什麼在「寫入資料庫之後」才發射事件？

這裡有一個重要的時序考量：**事件應該在資料已經成功寫入之後才發射**。如果我們在資料寫入之前就發射事件，其他 Service 可能會收到事件並試圖處理，但這筆資料實際上尚未存在——導致資料不一致。

所以，事件的發射順序永遠是：
1. 寫入本地資料庫 ✅
2. 發射事件 📡

### Post Service：發射 PostCreated 事件

```javascript
// 在 POST /posts 路由中
app.post('/posts', async (req, res) => {
  const id = crypto.randomBytes(4).toString('hex');
  const { title } = req.body;

  posts[id] = { id, title };

  // 在資料寫入後，向 Event Bus 發射事件
  await axios.post('http://localhost:4005/events', {
    type: 'PostCreated',
    data: { id, title }
  });

  res.status(201).send(posts[id]);
});
```

### Comment Service：發射 CommentCreated 事件

```javascript
// 在 POST /posts/:id/comments 路由中
app.post('/posts/:id/comments', async (req, res) => {
  const commentId = crypto.randomBytes(4).toString('hex');
  const { content } = req.body;
  const postId = req.params.id;

  // ... 寫入本地資料庫 ...

  // 在資料寫入後，向 Event Bus 發射事件
  await axios.post('http://localhost:4005/events', {
    type: 'CommentCreated',
    data: { id: commentId, content, postId }
  });

  res.status(201).send(comments);
});
```

### ⚠️ 目前會看到 404 錯誤——這是預期行為

當 Post Service 或 Comment Service 向 Event Bus 發射事件時，Event Bus 會嘗試將事件 POST 給所有 Service——但此時 Post Service、Comment Service 和 Query Service 都還沒有實作 `POST /events` 端點，所以會收到 404。

這不是錯誤，而是**正常的開發過程**。我們故意讓每個步驟分開進行，確保每個環節都能獨立驗證。

---

## 💡 重點摘要

- 事件的發射時機永遠在「本地資料庫寫入成功」之後，這是確保資料一致性的關鍵原則。
- `await axios.post(...)` 使用 `await` 是因為事件發射是一個網路請求，如果我們不等待它完成就回傳 response，會導致事件可能在資料真正寫入之前就發射出去。
- 目前 Event Bus 向其他 Service 發射事件時收到 404，是因為那些 Service 還沒有實作事件接收端點——這是完全正常的開發過渡狀態。
- 每個 Service 只需要知道 Event Bus 的 URL，不需要知道其他 Service 的 URL——這是 Event Bus 帶來的**關注點分離（Separation of Concerns）**。

## 關鍵字

PostCreated Event, CommentCreated Event, Event Emission, await, Event Bus, 404 Error, Asynchronous Event, Data Consistency, Separation of Concerns, POST /events
