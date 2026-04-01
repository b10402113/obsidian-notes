# 使用 Stan Publish 與 Subscribe 發布與接收事件

## 📝 課程概述

本單元實作如何利用 `node-nats-streaming` 的 `publish()` 與 `subscribe()` API，在 Publisher 與 Listener 之間完成第一個完整的事件發布與接收循環。我們會定義 Subject（頻道名稱）、理解 NATS 世界中「Message」與「Event」的術語對照，並動手完成基本的 Pub/Sub 流程。

---

## 核心觀念與實作解析

### 發布事件（Publisher）

```typescript
const data = JSON.stringify({
  id: '123',
  title: 'concert',
  price: 20
});

stan.publish('ticket:created', data, () => {
  console.log('Event published');
});
```

**`publish()` 的三個參數：**

1. **`'ticket:created'`** — Subject（頻道名稱），決定這個事件要發到哪個 Channel
2. **`data`** — 事件的實際內容，**必須是字串**（JSON 字串化後的結果）
3. **Callback（可選）** — 事件發送完成後的回調

> **重要限制**：NATS Streaming 只接受字串作為 Message 內容，**無法直接傳送 JavaScript 物件**。所有資料在發布前都必須先經過 `JSON.stringify()`，接收端再 `JSON.parse()` 回來。

---

### 接收事件（Listener）

```typescript
const subscription = stan.subscribe('ticket:created');

subscription.on('message', (msg) => {
  console.log('Message received');
});
```

**Subscribe 的兩個步驟：**

1. **`stan.subscribe('ticket:created')`** — 向 Server 表達「我想訂閱這個 Subject」
2. **`subscription.on('message', ...)`** — 設定收到訊息時的回調

> **這裡有個重要的設計差異**：很多事件庫是直接 `subscribe('channel', callback)` 把 Callback 當第二個參數傳入，但 `node-nats-streaming` 是**先建立 Subscription 物件，再在物件上監聽 `message` 事件**。這個差異在剛開始接觸時容易造成困惑。

---

### Subject 與 Channel 的關係

在 NATS 世界中：

- **Subject** = 發布事件時指定的頻道名稱
- **Channel** = NATS Streaming Server 內儲存事件流的地圖中的某個鍵
- **Subscription** = 某個 Client 向 Server 註冊的「我要接收這個 Subject 的事件」的請求

```
Publisher ──publish──▶ Subject: "ticket:created"
                              │
                    NATS Server 建立/寫入 Channel
                              │
                    Subscription 列表 ──▶ 各自的 Listener
```

---

### Message 物件的結構

當事件被送到 Listener 時，`msg`（Message 物件）並不是直接的資料本身，而是**一個包裝過的物件**，提供了以下重要方法：

| 方法 | 回傳值 |
|------|--------|
| `msg.getSubject()` | 這個訊息來自哪個 Subject |
| `msg.getSequence()` | 這個事件的**序列號**（第一個事件 = 1，第二個 = 2，依此類推） |
| `msg.getData()` | 事件實際攜帶的資料（字串或 Buffer） |

```typescript
subscription.on('message', (msg: Message) => {
  const data = msg.getData();
  if (typeof data === 'string') {
    console.log(`Received event #${msg.getSequence()} with data: ${data}`);
  }
});
```

---

### 序列號（Sequence Number）的重要性

每次在**同一個 Channel 上**發布事件時，NATS Streaming 都會自動為事件分配一個遞增的序列號。這個號碼在**解決並發問題**時會扮演關鍵角色——我們後續會看到如何利用序列號確保事件「只被處理一次」且「依正確順序處理」。

---

### 如果沒有人訂閱，發布會怎樣？

如果某個 Subject 沒有任何 Listener，呼叫 `publish()` **不會報錯**。Server 會如實發送，只是沒有人會收到。這在測試階段是正常的，但在正式系統中應該確保對應的 Listener 已啟動。

---

## 💡 重點摘要

- **`publish()` 只能接受字串，所有 JS 物件必須先 `JSON.stringify()`。**
- **`subscribe()` 是分兩步驟：先建立 Subscription 物件，再監聽 `message` 事件。**
- **Message 物件提供了 `getSequence()` 與 `getData()` 等方法，用來取出事件的元資料與內容。**
- **序列號是 NATS Streaming 內建的遞增編號，是後續解決並發問題的核心工具。**

---

## 🔑 關鍵字

publish, subscribe, Subject, Channel, Sequence, Message, getData, getSequence
