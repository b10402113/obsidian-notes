# Event Bus 事件同步與歷史補送實作

## 📝 課程概述

本單元實作了 Event Bus 的事件持久化（in-memory array）和 Query Service 的歷史事件拉取（Re-sync）機制。Event Bus 現在除了廣播事件，還會將每個收到的事件存入陣列，並提供 `GET /events` 端點讓任何 Service 在啟動時能一次取得所有歷史事件並處理——徹底解決了「Service 延後上線」時事件丟失的問題。

## 核心觀念與實作解析

### Event Bus 持久化事件

```js
// event-bus/index.js
const events = [];  // 事件持久化儲存

app.post('/events', (req, res) => {
  const event = req.body;
  events.push(event);  // 先存入，再廣播

  // 廣播給所有 Service...
  axios.post('http://localhost:4000/events', event);
  axios.post('http://localhost:4001/events', event);
  axios.post('http://localhost:4002/events', event);
  axios.post('http://localhost:4003/events', event);

  res.send({ status: 'OK' });
});

app.get('/events', (req, res) => {
  res.send(events);  // 提供歷史事件查詢
});
```

### 為什麼要先 `push` 再廣播？

如果先廣播後才存入，萬一存入前 Event Bus 當機，該事件就永久丟失了。先持久化再廣播，才能確保**At-least-once Delivery**（至少送達一次）。

### Query Service 啟動時拉取並處理歷史事件

```js
// query/index.js — 啟動後立即執行
app.listen(4002, async () => {
  console.log('Listening on 4002');

  // 向 Event Bus 取得所有歷史事件
  const res = await axios.get('http://localhost:4005/events');
  const events = res.data;

  for (let event of events) {
    console.log('Processing event:', event.type);
    handleEvent(event.type, event.data);
  }
});
```

### `handleEvent` 的重構

將 `POST /events` 中的事件處理邏輯提取為獨立函數，供「接收廣播」和「啟動時批量處理」兩處呼叫：

```js
// 提取為可復用函數
const handleEvent = (type, data) => {
  if (type === 'PostCreated') {
    posts[data.id] = { id: data.id, title: data.title, comments: [] };
  }
  if (type === 'CommentCreated') {
    posts[data.postId].comments.push({ id: data.id, content: data.content, status: data.status });
  }
  if (type === 'CommentUpdated') {
    const post = posts[data.postId];
    const comment = post.comments.find(c => c.id === data.id);
    comment.status = data.status;
    comment.content = data.content;
  }
};
```

### 完整測試流程

1. **Kill 掉 Query Service**（模擬延後上線）
2. **建立新文章和留言**（多個 `PostCreated`、`CommentCreated`、`CommentModerated`、`CommentUpdated` 事件進入 Event Bus 的 `events` 陣列）
3. **重新啟動 Query Service**
4. 觀察 Query Service Terminal：`Processing event: PostCreated` → `Processing event: CommentCreated` → ...
5. **開啟瀏覽器**：Query Service 完整呈現所有錯過期間的文章與留言

### 這個方案的剩餘限制

1. **in-memory 陣列**：重啟 Event Bus 後 `events` 陣列會消失。這些歷史事件需要存到**資料庫**（例如 PostgreSQL、MongoDB）才能真正持久化——這是 production 環境必須實作的部分。
2. **冪等性（Idempotency）**：若 `handleEvent` 處理同一個事件兩次，會產生重複資料（例如 `CommentCreated` 處理兩次，就會有兩筆相同的留言）。Production 系統需要在 `handleEvent` 內做「是否已處理過」的檢查。

## 💡 重點摘要

- **持久化 + 提供查詢端點**是 Event Bus 的核心改造——讓 Service 在任何時候都能補回錯過的事件。
- **冪等性**是事件驅動系統處理「重複事件」的必備機制，防止同一事件被多次處理造成資料重複。
- **in-memory 持久化**在 Event Bus 重啟後會消失——真實系統應使用資料庫（PostgreSQL、MongoDB）。
- 這套「Event Bus 儲存 + Service 上線時拉取並處理」的機制稱為 **Event Sourcing** 或 **Event Replay** 的簡化版本。

## 🔑 關鍵字

`Event Persistence`, `Event Replay`, `GET /events`, `At-least-once Delivery`, `Idempotency`, `Event Sourcing`
