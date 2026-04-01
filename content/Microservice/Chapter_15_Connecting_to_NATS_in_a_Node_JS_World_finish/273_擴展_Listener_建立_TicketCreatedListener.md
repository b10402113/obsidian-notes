# 擴展 Listener：建立 TicketCreatedListener

## 📝 課程概述

有了 `Listener` 抽象類之後，本單元示範如何建立第一個具體的子類——`TicketCreatedListener`。這個子類只定義三件事：`subject`、`queueGroupName` 與 `onMessage` 的商業邏輯，其餘繁瑣的訂閱設定全部由父類自動處理。這是 **Template Method Pattern** 的經典應用——父類定義骨架，子類填入具體細節。

---

## 核心觀念與實作解析

### 建立 TicketCreatedListener

```typescript
class TicketCreatedListener extends Listener {
  subject = 'ticket:created';           // 要監聽的 Channel
  queueGroupName = 'payments-service';  // Queue Group 名稱

  onMessage(data: any, msg: Message) {
    console.log('Event data:', data);
    msg.ack(); // 成功處理後明確回傳 ACK
  }
}
```

使用方式非常簡潔：

```typescript
const listener = new TicketCreatedListener(client);
listener.listen();
```

### 程式碼重構：將 Listener 獨立成檔案

授課講師將程式碼重新組織為以下結構：

```
events/
  BaseListener.ts      ← 抽象類 Listener
  TicketCreatedListener.ts ← 子類
listener.ts            ← 測試用的啟動檔案
publisher.ts           ← 發布事件的測試檔案
```

### 為什麼要手動呼叫 msg.ack()？

NATS 的 Manual Ack Mode 核心邏輯是：

- **商業邏輯執行成功** → 呼叫 `msg.ack()` → 告知 NATS「這條訊息已處理，可以移除」
- **商業邏輯執行失敗** → **不呼叫 `ack()`** → NATS 等待 `ackWait` 時間後自動重新投遞訊息

> 這個設計非常優雅——**我們不需要任何 try/catch 或額外邏輯**。只要省略 `ack()` 呼叫，NATS 就會自動重試。這也是 Event-Driven Architecture 中「失敗即重試」原則的體現。

### 程式碼組織策略

| 檔案 | 歸屬 |
|---|---|
| `BaseListener.ts` | 將來放在 `common` 模組，供所有 Service 引用 |
| `TicketCreatedListener.ts` | 放在各 Service 內部（各 Service 處理邏輯不同） |

---

## 💡 重點摘要

- **子類只需定義三件事**：`subject`、`queueGroupName`、`onMessage`，其餘由父類處理。
- **`msg.ack()` 的哲學是「成功才確認，失敗自動重試」**，不需要複雜的錯誤處理。
- **`TicketCreatedListener` 屬於 Service 內部實作，`BaseListener` 屬於共享模組**——這個分層非常重要。

---

## 🔑 關鍵字

TicketCreatedListener, Template Method Pattern, Manual Ack Mode, Queue Group, common module
