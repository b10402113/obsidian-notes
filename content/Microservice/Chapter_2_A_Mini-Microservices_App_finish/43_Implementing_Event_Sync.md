# Event Sync 實作：Event Bus 事件儲存與 Query Service 啟動同步

## 📝 課程概述

本單元實作 Option 3 的核心功能。我們會改造 Event Bus，讓它在收到每個事件的同時將其存入一個事件陣列，並新增一個 `GET /events` 端點供其他 Service 查詢歷史事件。同時，Query Service 在啟動時會自動向 Event Bus 請求所有歷史事件，並逐一 replay 處理，讓自己的本地資料與系統真實狀態完全同步。

---

## 核心觀念與實作解析

### Event Bus 的改造：儲存所有事件

```javascript
const events = []; // 事件儲存

app.post('/events', (req, res) => {
  const event = req.body;
  events.push(event); // 先儲存
  // 然後廣播給所有 Service
  axios.post('http://localhost:4000/events', event);
  axios.post('http://localhost:4001/events', event);
  axios.post('http://localhost:4002/events', event);
  axios.post('http://localhost:4003/events', event);
  res.send({});
});

// GET /events：讓 Service 主動拉取所有歷史事件
app.get('/events', (req, res) => {
  res.send(events);
});
```

> 這裡用 in-memory 陣列儲存，生產環境會用真正的資料庫（PostgreSQL、MongoDB 等）來持久化。

### Query Service 的啟動同步

在 Query Service 啟動時（`app.listen` 的 callback 中），主動同步：

```javascript
const handleEvent = (type, data) => {
  // 將所有事件處理邏輯集中在這裡
  if (type === 'PostCreated') { /* ... */ }
  if (type === 'CommentCreated') { /* ... */ }
  if (type === 'CommentUpdated') { /* ... */ }
};

app.listen(4002, async () => {
  console.log('Listening on 4002');

  // 啟動時向 Event Bus 請求所有歷史事件
  const res = await axios.get('http://localhost:4005/events');
  for (let event of res.data) {
    handleEvent(event.type, event.data);
  }
});
```

### 重構：提取 handleEvent 為可復用函式

為什麼要提取 `handleEvent` 函式？
- Event Bus 廣播過來的事件 → 需要處理
- Query Service 啟動時同步的歷史事件 → 也需要處理

這兩種場景的處理邏輯完全相同，所以提取成一個共享函式。

### 端對端測試流程

1. **停止 Query Service**
2. **瀏覽器建立新文章 + 留言**（此時 Event Bus 儲存了這些事件）
3. **Query Service 恢復上線** → 自動向 Event Bus GET `/events` → replay 所有事件 → 本地資料同步完成
4. **刷新瀏覽器** → 正常顯示所有資料

這個流程完美解決了「Service 後啟動就拿不到歷史資料」的問題。

---

## 💡 重點摘要

- Event Bus 現在成為了**系統的單一事件來源（Single Source of Events）**，所有事件都被持久化儲存。
- Query Service 的 `handleEvent` 函式是整個 replay 機制的核心——無論事件來自即時廣播還是歷史同步，處理邏輯完全相同。
- 這個模式稱為 **Event Sourcing**：系統的狀態是通過 replay 所有歷史事件而來的，而非直接存儲「當前狀態」。
- 生產環境會使用 Kafka 或 NATS 等 Message Broker，它們內建了事件持久化與 replay 機制，不需要像我們這樣手工實作。

## 關鍵字

Event Store, GET /events, handleEvent Extraction, Event Replay, Service Startup Sync, Event Sourcing, Event Bus Persistence, Query Service, History Sync, Single Source of Events, In-Memory Events Array
