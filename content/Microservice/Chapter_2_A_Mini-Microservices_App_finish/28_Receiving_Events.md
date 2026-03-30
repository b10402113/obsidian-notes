# 事件接收端點：Post Service、Comment Service 與 Query Service 骨架實作

## 📝 課程概述

本單元完成兩件事：首先，讓 Post Service 與 Comment Service 都具備接收事件的能力（`POST /events` 端點），這樣 Event Bus 向它們廣播事件時才不會收到 404。其次，我們開始搭建第三個 Service——**Query Service**（Listen on Port 4002），它將負責儲存「已組合好的」Posts + Comments 資料，供 React App 一次取得所有需要的前端資料。

---

## 核心觀念與實作解析

### 為什麼 Post Service 和 Comment Service 也要接收事件？

目前的設計中，只有 Query Service 需要處理事件。但從架構的角度來說，所有 Service 都應該具備接收事件的端點，因為：
- 確保 Event Bus 的廣播不會因某個 Service 無法接收而中斷
- 為未來的功能擴展預留空間（例如：Post Service 未來可能也需要對某些事件做出反應）

### Post Service / Comment Service 的事件接收端點

```javascript
app.post('/events', (req, res) => {
  console.log('Received Event:', req.body.type);
  res.send({});
});
```

目前這兩個 Service 收到事件後，只是 `console.log` 一下，並不做任何處理——這是刻意為之的設計，它們**目前並不關心**別的 Service 發出了什麼事件。

### Query Service 骨架

Query Service 是第三個 Express 應用程式，Listen on Port 4002。它有兩個 Route：

| Method | Path | 功能 |
|--------|------|------|
| `GET` | `/posts` | 回傳所有文章及其留言的組合資料 |
| `POST` | `/events` | 接收並處理來自 Event Bus 的事件 |

Query Service **不需要安裝 Axios**，因為它只接收事件，不發射事件。

---

## 💡 重點摘要

- 所有 Service 都應該實作 `POST /events` 端點，即使目前不需要處理任何事件——這是 Event Bus 順利廣播的前提。
- Query Service 的 `GET /posts` 是 React App 未來唯一的資料來源，大幅簡化了前端邏輯。
- Query Service 目前處於「骨架」狀態——事件處理邏輯尚未實作，這是下一個單元的重點。

## 關鍵字

POST /events, Query Service, Event Receiver, Service Skeleton, GET /posts, In-Memory Storage, Port 4002, Express Boilerplate
