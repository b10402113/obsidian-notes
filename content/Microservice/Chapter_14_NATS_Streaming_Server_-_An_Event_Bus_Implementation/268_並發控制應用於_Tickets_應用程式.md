# 並發控制應用於 Tickets 應用程式

## 📝 課程概述

本單元將 Transaction Service 的並發控制思路，實際映射到我們的 Tickets 應用程式上。我們會說明 Ticket Service 如何在**每次修改 Ticket 時遞增 Version 欄位**，並讓其他依賴 Ticket 資料的 Service（如 Orders Service）透過這個 Version 欄位，確保只處理「已按順序」的更新事件。

---

## 核心觀念與實作解析

### Ticket Service 的資料模型改動

每個 Ticket 記錄除了 `id`、`title`、`price` 等欄位外，還需要增加一個 **`version`** 欄位：

```json
{
  "id": "ticket_abc",
  "title": "Concert",
  "price": 10,
  "version": 1   // 第 1 次建立
}
```

**每次 Ticket 被更新時，version 都會遞增：**

```
建立 Ticket → version = 1
更新 Price → version = 2
再次更新   → version = 3
```

---

### 事件結構（Ticket Service 發布到 NATS）

```json
{
  "type": "ticket:updated",
  "id": "ticket_abc",
  "price": 50,
  "version": 2
}
```

> **關鍵區別**：Version 是由 **Ticket Service（canonical source of truth）** 主動管理的，不是由 NATS Streaming 分配。我們不需要依賴 NATS 的序列號——Ticket Service 的資料庫就是自己的序列號來源。

---

### Orders Service 如何處理並發問題

Orders Service 負責維護一個「Ticket 快照」資料表，每次收到事件時，透過 Version 欄位確保依序處理：

```typescript
subscription.on('message', (msg: Message) => {
  const { id, price, version } = JSON.parse(msg.getData());

  const snapshot = getTicketSnapshot(id);

  // 關鍵檢查：version 是否為 lastVersion + 1
  if (version !== snapshot.lastVersion + 1) {
    // 版本號對不上，說明前面的更新還沒處理完
    // 不要 ack，等待 Server 重送
    return;
  }

  // 版本號對上了，正常處理
  snapshot.price = price;
  snapshot.lastVersion = version;
  save(snapshot);
  msg.ack();
});
```

---

### 為什麼 Version 欄位比 NATS 序列號更可靠？

| 維度 | NATS 序列號 | Ticket Service Version 欄位 |
|------|------------|--------------------------|
| 誰管理 | NATS Streaming Server | Ticket Service 資料庫 |
| 是否跨 Service 共享 | 所有 Channel 共享同一遞增號 | 每個 Ticket 獨立遞增 |
| 重新啟動 Server 後 | 序列號從頭開始（視設定） | Version 永遠跟隨資料庫記錄 |
| 是否受網路延遲影響 | 會（NATS 分配） | 不會（資料庫寫入後就確定） |

> **核心原因**：NATS 的序列號是**全域**遞增的（一個 Channel 上的事件），而 Version 是**每筆記錄**獨立遞增。當一個 Service 只負責一個類型的資源（如 Ticket Service 負責 Ticket），它的 Version 欄位就是一個**完全自治的序列號**。

---

### MongoDB 與 Mongoose 的內建支援

> MongoDB + Mongoose 提供了內建的 `versionKey`（又稱 `__v`）功能，可以自動管理文件版本。這讓我們不需要手動維護 Version 欄位，Mongoose 會自動在每次 `save()` 時遞增該欄位。

但理解背後的原理仍然重要——否則當你需要自訂版本檢查邏輯（例如跨文件的版本控制）時，將無從下手。

---

### 這個方法解決了哪些並發問題？

✅ **事件處理失敗後重送** — Version 不連貫的事件會被自動忽略
✅ **事件亂序到達** — 搶先到達的事件因 Version 不符合 `lastVersion + 1` 而被拒絕
✅ **多實例處理速度差異** — Queue Group 只確保發給一個實例，Version 控制確保依序處理
✅ **跨用戶隔離** — 每個 Ticket 有獨立的 Version，不同 Ticket 的更新不會互相影響

---

## 💡 重點摘要

- **在 Ticket 記錄中加入 `version` 欄位，每次更新時遞增，由 Ticket Service 的資料庫管理。**
- **事件攜帶 `version`，Consumer Service 透過檢查 `version === lastVersion + 1` 確保依序處理。**
- **Version 比 NATS 序列號更適合控制並發，因為它是每筆記錄獨立遞增的。**
- **MongoDB/Mongoose 內建 `versionKey` 功能可以自動管理版本欄位，大幅降低實作負擔。**

---

## 🔑 關鍵字

Version, versionKey, Ticket Service, Orders Service, Optimistic Locking, Event Ordering, Mongoose
