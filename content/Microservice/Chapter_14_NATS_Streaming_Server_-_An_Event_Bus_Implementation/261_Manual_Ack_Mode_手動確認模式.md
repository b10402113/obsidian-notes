# Manual Ack Mode：手動確認模式

## 📝 課程概述

本單元說明 NATS Streaming 預設的「自動確認（Auto Ack）」行為，以及為什麼在大多數生產場景中，我們需要關閉自動確認，改用**手動確認（Manual Ack）**模式。我們會學習 `setManualAckMode()` 的設定方式，以及 `msg.ack()` 的呼叫時機，並透過實際演示展示如果忘記呼叫 `ack()` 會發生什麼事。

---

## 核心觀念與實作解析

### 預設行為：Auto Ack（自動確認）

在預設設定下，一旦事件被送到 Subscription 的 `message` 事件處理器，**即使程式碼發生錯誤或尚未完成處理**，NATS Streaming 也會自動向 Server 發出「已收到並處理完成」的確認訊號。

> **問題所在**：如果我們在處理事件的過程中發生了錯誤（比如資料庫連線失敗），Server 會認為「事件已被成功處理了」，從而**不再重送這個事件**，寶貴的資料就此遺失。

---

### 為什麼需要 Manual Ack Mode？

```typescript
// 設定 Subscription Options
const options = stan.subscriptionOptions();
options.setManualAckMode(true);  // 關閉自動確認

const subscription = stan.subscribe('ticket:created', 'orders-service', options);

subscription.on('message', (msg: Message) => {
  // 處理事件...
  // （假設這裡發生了錯誤）

  // 處理成功後，手動呼叫 ack()
  msg.ack();
});
```

**設定流程：**

1. `stan.subscriptionOptions()` — 建立 Subscription 選項物件
2. `options.setManualAckMode(true)` — 開啟手動確認模式
3. 在訊息處理**成功完成後**，明確呼叫 `msg.ack()`

---

### 忘記呼叫 `ack()` 會發生什麼？

Server 在發送事件後會**等待 ack 回覆**。如果等待 **30 秒（預設值）** 後仍沒有收到確認，Server 就會**重新發送事件**給同一個 Queue Group 中的另一個實例：

```
Event #59 發送給 Listener A
  │
  │  30 秒後...
  │
  ▼  沒有收到 ack
Server 重新發送 Event #59 給 Listener B
  │
  │  又 30 秒後...
  │
  ▼  還是沒收到 ack
Server 再次發送 Event #59 給 Listener A（或下一個）
  ...
```

這就是 NATS Streaming 的**自動重送（Auto-redelivery）**機制，也是為什麼 Manual Ack Mode 如此重要的原因。

---

### Manual Ack Mode 的實作模式

```typescript
subscription.on('message', (msg: Message) => {
  const data = JSON.parse(msg.getData() as string);

  try {
    // 嘗試處理事件（例如寫入資料庫）
    await processEvent(data);

    // 處理成功，發送 ack
    msg.ack();
  } catch (err) {
    // 處理失敗，不呼叫 ack
    // 事件會在 30 秒後自動重送
    console.error('Processing failed, will retry...');
  }
});
```

> **最佳實踐**：所有可能失敗的 I/O 操作（資料庫寫入、網路請求等）都應該放在 `try/catch` 區塊中處理，確保只有「真正成功完成」的事件才會呼叫 `ack()`。

---

## 💡 重點摘要

- **預設的 Auto Ack 模式會導致處理失敗時事件遺失——Server 不知道我們其實沒有處理成功。**
- **開啟 Manual Ack Mode (`setManualAckMode(true)`) 後，必須在成功完成後明確呼叫 `msg.ack()`。**
- **如果 30 秒內沒有收到 ack，Server 會自動將事件重新發送給 Queue Group 中的另一個實例。**
- **所有資料庫寫入等 I/O 操作都應該包在 `try/catch` 中，確保只有真正成功的處理才發 ack。**

---

## 🔑 關鍵字

Manual Ack, Auto Ack, msg.ack, Redelivery, 30 Seconds, Auto-redelivery, subscriptionOptions
