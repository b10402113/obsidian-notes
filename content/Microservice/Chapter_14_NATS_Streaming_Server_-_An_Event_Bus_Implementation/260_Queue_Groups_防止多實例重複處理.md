# Queue Groups：防止多實例重複處理

## 📝 課程概述

本單元介紹 Queue Group 這個核心概念，並說明為什麼當同一個 Service 有**多個實例（Instance）**同時運行時，事件不應該同時發給所有實例。我們會透過一個真實的資料庫重複寫入問題，說明 Queue Group 的使用場景與設定方式。

---

## 核心觀念與實作解析

### 問題：當同一個 Service 有多個實例時

假設我們有兩個 Order Service 實例，同時收到一筆 `ticket:created` 事件。如果不做任何控制，**兩個實例都會處理這筆事件**：

```
Event: ticket:created
  ├──▶ Order Service A → 處理：新增一筆記錄到資料庫
  └──▶ Order Service B → 處理：新增**同樣**一筆記錄到資料庫
```

> **後果**：同一筆事件被處理兩次，資料庫中出現**重複的錯誤資料**。如果這個事件是「建立訂單」，使用者可能會收到兩張相同的訂單。

---

### Queue Group 的解決方案

```typescript
// 第二個參數就是 Queue Group 名稱
const subscription = stan.subscribe('ticket:created', 'orders-service');
```

設定 Queue Group 後，NATS Streaming Server 的行為變成：

1. 同一個 Queue Group 內的所有實例**共享一個佇列**
2. **同一筆事件只會被發給群組中的其中一個實例**（Server 會輪流選擇）
3. 其他不在 Queue Group 中的訂閱者，依然會收到事件

```
Channel: ticket:created
  │
  ├── [Queue Group: orders-service]
  │      ├──▶ Order Service A   ✅ 收到（輪到這次）
  │      └──▶ Order Service B   ❌ 這次不發送
  │
  └── (無 Queue Group 的 Listener)
         └─▶ Query Service      ✅ 也會收到
```

---

### Queue Group 的命名慣例

Queue Group 名稱可以自由設定，**只要是字串即可**。常見的命名方式是直接以 Service 名稱命名：

```typescript
stan.subscribe('ticket:created', 'orders-service-queue-group');
```

> **關鍵規則**：所有屬於**同一個 Service** 的實例，在訂閱時必須使用**完全相同的 Queue Group 名稱**。這樣 NATS Server 才會知道要將它們視為同一組，確保每個事件只發給其中一個實例。

---

### 兩個實例收到事件的分配模式

Server 並不總是完美輪流（Round-robin）。在某些情況下，可能連續多次都發給同一個實例。這是 Server 內部的實作細節，原則上我們只需要確保：**所有同一 Service 的實例都加入同一個 Queue Group，就能避免重複處理**。

---

### 實作設定方式

在 `stan.subscribe()` 的**第二個參數**（也就是原本的 Queue Group 名稱位置）填入字串：

```typescript
// 在訂閱時加入 Queue Group 名稱
const subscription = stan.subscribe('ticket:created', 'orders-service-queue-group');

subscription.on('message', (msg: Message) => {
  console.log('Event received!');
  // 處理事件...
});
```

這個參數**預設是空字串（相當於無 Queue Group）**。只要傳入非空字串，該 Subscription 就會被加入指定的 Queue Group。

---

## 💡 重點摘要

- **Queue Group 解決的核心問題：同一個 Service 的多個實例，不應該同時處理同一個事件，否則會導致資料庫重複寫入。**
- **Queue Group 是以「名稱」區分的字串，同一 Service 的所有實例必須使用相同的名稱。**
- **有 Queue Group 的情況下，Server 將事件發給群組內的其中一個實例；沒有 Queue Group 的其他訂閱者依然會收到事件。**
- **Queue Group 名稱的命名慣例：以 Service 名稱命名（例如 `orders-service`）。**

---

## 🔑 關鍵字

Queue Group, Duplicate Processing, Subscribe, Round-robin, Service Instance, Shared Queue
