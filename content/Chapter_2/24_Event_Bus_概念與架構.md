# Event Bus 概念與架構

## 📝 課程概述

本單元引入了 **Event Bus**（事件匯流排）——這是我們自己手刻的訊息中介層，位於所有 Service 之間。當任何一個 Service 需要通知其他 Service「某件事發生了」，它就把事件發到 Event Bus，Event Bus 再廣播給所有訂閱者。本單元先介紹概念，下個單元再實際建構程式碼。

## 核心觀念與實作解析

### Event Bus 的核心職責

```
┌──────────────┐    POST /events     ┌──────────────┐
│  Posts Svc   │ ──────────────────→ │              │
└──────────────┘                     │              │
                                      │  Event Bus   │──→ Posts Service
┌──────────────┐    POST /events     │  (Port 4005) │──→ Comments Service
│ Comments Svc │ ──────────────────→│              │──→ Query Service
└──────────────┘                     │              │──→ Moderation Svc
                                      └──────────────┘
```

Event Bus 只有**一個端點**：`POST /events`。任何 Service 把事件 body 發過去，Event Bus 就把同一份 payload 用 `POST /events` 轉發給**所有其他正在運行的 Service**。

### 為什麼要自己手刻 Event Bus？

業界有成熟的解決方案，如 **RabbitMQ、Kafka、NATS**。但手刻能讓讀者理解背後的訊息流動邏輯。老師強調：本專案的 Event Bus **不是 production-ready 的**，它缺少：
- 訊息持久化（重啟丟事件）
- 錯誤重試機制
- 訂閱過濾（並非所有 Service 都需要接收所有事件）

> 在後期的大型專案中，會使用真正的 Event Bus（例如 NATS）。

### Event 的結構設計

```js
// 所有事件都統一使用這種結構
{
  type: 'PostCreated',        // 事件類型（字串）
  data: {                    // 事件攜帶的資料
    id: 'abc123',
    title: 'My Post'
  }
}
```

- **`type`**：事件的種類名稱，接收端依此判斷「該怎麼處理這個事件」。
- **`data`**：與事件相關的具體資料，結構可自由定義（但原則上同一個 `type` 的 `data` 結構應保持一致）。

### Event Bus 的實作邏輯

```js
// event-bus/index.js
app.post('/events', (req, res) => {
  const event = req.body;  // 接收來自某個 Service 的事件

  // 廣播給所有 Service
  axios.post('http://localhost:4000/events', event);  // posts
  axios.post('http://localhost:4001/events', event);  // comments
  axios.post('http://localhost:4002/events', event);  // query

  res.send({ status: 'OK' });
});
```

每個 Service 收到 `POST /events` 後，各自根據 `event.type` 來決定是否要處理這個事件（`posts` 與 `comments` 對 `PostCreated` 事件不感興趣，僅做 `console.log`）。

## 💡 重點摘要

- **Event Bus 是所有 Service 之間的訊息中樞**，它實現了「發送者不需要知道誰在接收」的完全去耦合。
- 事件結構統一使用 `{ type, data }` 是業界常見的慣例，讓接收端能以 `switch/if` 簡單處理。
- 手刻 Event Bus 的目的是**理解原理**，production 環境應使用 RabbitMQ / Kafka / NATS 等成熟框架。
- Event Bus 缺少訊息持久化——服務重啟後遺失的未處理事件無法復原，這是本實作的重大限制。

## 🔑 關鍵字

`Event Bus`, `Pub/Sub`, `RabbitMQ`, `Kafka`, `NATS`, `Event Broadcasting`
