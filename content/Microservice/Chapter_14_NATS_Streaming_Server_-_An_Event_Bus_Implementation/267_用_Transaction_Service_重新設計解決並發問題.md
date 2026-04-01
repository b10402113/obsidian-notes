# 用 Transaction Service 重新設計：解決並發問題的關鍵

## 📝 課程概述

本單元是並發問題章節的**核心解答**。老師提出一個關鍵洞見：**問題的根源不是 Event Bus 的設計不足，而是我們對「發布事件的 Service」的設計有缺陷**。透過引入一個 Transaction Service（負責記錄所有交易並為其編號），再讓其他 Service 根據事件中的交易編號決定是否處理，就能優雅地解決所有並發問題。

---

## 核心觀念與實作解析

### 為什麼之前的所有方案都失敗了？

之前的方案都試圖**在 Event Bus 層面強行解決問題**。但正確的思路應該是：**從「產生事件的 Service」的資料模型入手**。

> 老師的關鍵洞見：**不能依賴 NATS Server 的序列號來控制並發，而是要在事件來源資料庫中為每筆「意圖」建立自己的編號。**

---

### Transaction Service 的設計

```
使用者請求流程：
  1. 使用者按下「存款 $70」
  2. Request → Transaction Service
  3. Transaction Service 將交易寫入自己的資料庫（包含唯一的 transactionId 與 transactionNumber）
  4. Transaction Service 發布事件到 NATS Streaming
```

**Transaction Service 的資料庫記錄：**

```json
[
  { "id": "tx001", "userId": "cziq", "type": "deposit", "amount": 70,  "transactionNumber": 1 },
  { "id": "tx002", "userId": "cziq", "type": "deposit", "amount": 40,  "transactionNumber": 2 },
  { "id": "tx003", "userId": "cziq", "type": "withdraw","amount": 100, "transactionNumber": 3 }
]
```

**事件結構（Transaction Service 發布到 NATS）：**

```json
{
  "type": "transaction:created",
  "transactionId": "tx001",
  "userId": "cziq",
  "amount": 70,
  "transactionNumber": 1
}
```

---

### 事件攜帶 transactionNumber 的意義

```typescript
// Account Service 的處理邏輯
subscription.on('message', (msg: Message) => {
  const { userId, amount, transactionNumber } = JSON.parse(msg.getData());

  const userAccount = getAccountFromDB(userId);

  // 關鍵：只有當 transactionNumber = 上一個處理的號碼 + 1 時，才處理
  if (transactionNumber === userAccount.lastTransactionNumber + 1) {
    userAccount.balance += amount;
    userAccount.lastTransactionNumber = transactionNumber;
    save(userAccount);
    msg.ack();
  } else {
    // 號碼不對（前面的交易還沒處理完），不 ack，等 Server 重送
    // 不要做任何事
  }
});
```

---

### 為什麼這能解決所有並發問題？

**情境一：事件處理失敗**

```
Event #1 (deposit $70) → Service B 處理
  失敗（DB 鎖住）
  不 ack，Server 30 秒後重送給 Service A

Event #2 (deposit $40) → Service A 收到
  檢查：lastTransactionNumber = undefined ≠ 1 ✅
  等待（不 ack）

Event #1 重送 → Service A 收到
  檢查：lastTransactionNumber = undefined = 0 + 1 ✅
  處理：餘額 = $70，lastTransactionNumber = 1，ack

Event #2 重送 → Service A 收到
  檢查：lastTransactionNumber = 1 = 1 + 1 ✅
  處理：餘額 = $110，lastTransactionNumber = 2，ack
```

**情境二：事件亂序到達**

```
Event #2 先到 → 檢查：lastTransactionNumber = undefined ≠ 1 → 拒絕，不 ack
Event #1 到   → 檢查：lastTransactionNumber = undefined = 0 + 1 ✅ → 處理
Event #2 重送 → 檢查：lastTransactionNumber = 1 = 1 + 1 ✅ → 處理
```

**情境三：不同用戶完全隔離**

```
Jim 的 Event #3 失敗 → 只影響 Jim，不影響 Mary 的存款
因為每個用戶在 Account Service 資料庫中有自己的 lastTransactionNumber
```

---

### 設計核心原則

> **交易的意圖（Intent）與交易編號必須來自於一個唯一的、擁有該資源的 Service。** 這個 Service 必須在「發布事件之前」就已經將交易寫入自己的資料庫。Event Bus 只是通知下游的管道，**真正的資料來源是 Transaction Service 的資料庫**。

---

## 💡 重點摘要

- **並發問題的解決關鍵不在 Event Bus，而在於「發布事件的 Service」的資料模型設計。**
- **Transaction Service 在發布事件前，就先將交易寫入資料庫並產生 transactionNumber，讓事件攜帶「自己」的編號。**
- **Consumer Service 透過檢查 transactionNumber 是否為 lastTransactionNumber + 1，確保依序處理，並自動忽略過早到達的事件。**
- **每個用戶（或資源）都有獨立的 lastTransactionNumber，不同用戶之間完全隔離，不會互相阻塞。**

---

## 🔑 關鍵字

Transaction Service, transactionNumber, lastTransactionNumber, Intent, Sequential Processing, Resource Isolation
