# Moderation Service 與 CommentUpdated 完整實作流程

## 📝 課程概述

本單元完成整個 Moderation 功能的事件處理閉環。我們會實作 Moderation Service 的 `CommentCreated` 處理邏輯、Comment Service 接收 `CommentModerated` 並發出 `CommentUpdated` 通用事件的流程，以及 Query Service 接收 `CommentUpdated` 並更新本地資料的邏輯。這些環節組合起來，就是一個完整的 Domain-Owned Event Pattern 實作。

---

## 核心觀念與實作解析

### Moderation Service：處理 CommentCreated

```javascript
app.post('/events', async (req, res) => {
  const { type, data } = req.body;

  if (type === 'CommentCreated') {
    const status = data.content.includes('orange') ? 'rejected' : 'approved';

    await axios.post('http://localhost:4005/events', {
      type: 'CommentModerated',
      data: { id: data.id, content: data.content, postId: data.postId, status }
    });
  }

  res.send({});
});
```

### Comment Service：接收 CommentModerated，發出 CommentUpdated

Comment Service 是 Comment 的 Domain Owner，因此只有它知道如何處理 `CommentModerated` 這個專業更新事件：

```javascript
if (type === 'CommentModerated') {
  const { id, postId, status } = data;
  const comments = commentsByPostId[postId];
  const comment = comments.find(c => c.id === id);
  comment.status = status; // 直接修改記憶體中的物件

  // 發出通用更新事件
  await axios.post('http://localhost:4005/events', {
    type: 'CommentUpdated',
    data: { id, content: comment.content, postId, status }
  });
}
```

> 直接修改物件屬性即可，不需要重新 `push` 回陣列——因為物件是 reference，修改後記憶體中的狀態已經更新。

### Query Service：接收 CommentUpdated

```javascript
if (type === 'CommentUpdated') {
  const { id, content, postId, status } = data;
  const post = posts[postId];
  const comment = post.comments.find(c => c.id === id);
  // 用 event 提供的所有 attributes 直接覆蓋
  Object.assign(comment, { id, content, status });
}
```

### ⚠️ 別忘了更新 Event Bus 的廣播清單

當 Moderation Service 建立後，Event Bus 還沒有把它加入廣播清單。我們必須在 Event Bus 的 `POST /events` handler 中新增一行：

```javascript
axios.post('http://localhost:4003/events', event); // Moderation Service
```

否則其他 Service 的事件永遠不會被廣播給 Moderation Service。

---

## 💡 重點摘要

- `CommentModerated` 是專業的 Domain Event，只有 Comment Service（Domain Owner）有權處理它。
- `CommentUpdated` 是通用更新事件，其他 Service 只需要「接收並套用更新」，不需要理解背後的業務邏輯。
- 直接修改物件屬性後不需要重新 `push` 回陣列——JavaScript 的物件是 reference，改了就是改了。
- Event Bus 的廣播清單需要手動維護，每次新增 Service 都必須記得更新。

## 關鍵字

CommentModerated, CommentUpdated, Domain Owner, Object.assign, Event Processing, Comment Service, Query Service, Moderation Service, Status Update, Event Broadcasting
