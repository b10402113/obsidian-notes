# 事件資料的存取與 Message API

## 📝 課程概述

本單元深入探討 `node-nats-streaming` 的 Message 物件，說明如何從接收到的訊息中取出 Subject、序列號與實際資料（Data），並解釋 NATS 世界中「Message」與「Event」這兩個術語的對應關係。同時也會展示 TypeScript 型別的處理方式。

---

## 核心觀念與實作解析

### Message 物件的完整 API

`node-nats-streaming` 的 `Message` 介面提供了多個方法：

```typescript
import { Message } from 'node-nats-streaming';

subscription.on('message', (msg: Message) => {
  // 取出 Subject 名稱
  const subject = msg.getSubject();    // 回傳字串

  // 取出序列號（第一個事件 = 1）
  const sequence = msg.getSequence();  // 回傳數字

  // 取出實際資料
  const rawData = msg.getData();       // 回傳 string | Buffer

  console.log(`Subject: ${subject}, Seq: ${sequence}, Data: ${rawData}`);
});
```

**注意**：`getData()` 的回傳型別是 `string | Buffer`，在 TypeScript 中我們需要做型別檢查：

```typescript
const data = msg.getData();
if (typeof data === 'string') {
  const parsed = JSON.parse(data);
  console.log(parsed);
}
```

---

### 為什麼 `getData()` 可能回傳 Buffer？

NATS Streaming 在底層傳輸時，資料可能被 Node.js 的 Buffer 物件包裝。如果我們確定自己傳送的是純文字 JSON，就手動將其轉為字串處理。

---

### 序列號（Sequence Number）的意義

NATS Streaming 會為**每個 Channel 上的事件**維護一個遞增序號：

```
Channel: ticket:created
  Event #1  →  "ticket created" (id: abc, price: 10)
  Event #2  →  "ticket updated" (id: abc, price: 50)
  Event #3  →  "ticket updated" (id: abc, price: 100)
```

這個序號可以用來：

1. **確認事件是否已被處理過**（避免重複處理）
2. **驗證事件的先後順序**（用序號差異判斷事件是否亂序到達）
3. **作為並發控制的基礎**（類似樂觀鎖的 Version 概念）

這些應用我們會在後續的並發控制章節中看到完整的實作。

---

### 術語對照

| 課程中的說法 | NATS 官方術語 |
|------------|-------------|
| Event（事件） | Message（訊息） |
| Subject（頻道名） | Subject（頻道名稱） |
| 發布 | Publish |
| 訂閱 | Subscribe |

---

## 💡 重點摘要

- **`getData()` 的回傳型別是 `string | Buffer`，需要型別檢查才能安全地 `JSON.parse()`。**
- **序列號是 Channel 維度的遞增整數，是後續實現「只處理一次」與「依序處理」的關鍵工具。**
- **在 NATS 生態中，「事件」通常稱為 Message，請在閱讀官方文件時注意這個術語差異。**

---

## 🔑 關鍵字

Message, getData, getSequence, getSubject, TypeScript, Buffer, Sequence Number
