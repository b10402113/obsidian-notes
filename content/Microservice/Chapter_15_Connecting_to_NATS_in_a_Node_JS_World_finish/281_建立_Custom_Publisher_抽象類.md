# 建立 Custom Publisher 抽象類

## 📝 課程概述

與 `Listener` 抽象類相對應，本單元為 NATS 的 **Publisher 端**建立同樣的 Generic 抽象類 `Publisher`。目標一致：**確保 Subject 名稱與發布的資料結構在編譯期就完全匹配**，杜絕「發送了一個不存在屬性的 Event」這類微服務中最常見的 Runtime 錯誤。

---

## 核心觀念與實作解析

### BasePublisher 的設計

```typescript
interface Event {
  subject: Subjects;
  data: any;
}

export abstract class Publisher<T extends Event> {
  abstract subject: T['subject'];
  protected client: Stan;

  publish(data: T['data']) {
    this.client.publish(
      this.subject,
      JSON.stringify(data)   // ⚠️ 易忘：需手動 JSON.stringify
    );
  }
}
```

### 與 Listener 的設計對稱性

| 角色 | 類別 | Generic 約束 |
|---|---|---|
| 接收事件 | `Listener<T extends Event>` | `subject` 自動對齊 `T['subject']` |
| 發布事件 | `Publisher<T extends Event>` | `subject` 自動對齊 `T['subject']` |

### TicketCreatedPublisher 的實作

```typescript
class TicketCreatedPublisher extends Publisher<TicketCreatedEvent> {
  subject = Subjects.TicketCreated;
}

// 使用方式
const publisher = new TicketCreatedPublisher(client);
publisher.publish({ id: '123', title: 'concert', price: 20 });
//                             ↑ TypeScript 自動檢查屬性完整性
```

### Subject 的一致性保證

```typescript
subject: Subjects.TicketCreated = Subjects.TicketCreated;
```

與 Listener 同理——explicit type annotation + value 兩者並用，確保工程師無法將 `subject` 替換成其他 Subject。

> **Publisher 端的錯誤比 Listener 端更危險**：Listener 收到錯誤資料頂多處理失敗；但 Publisher 發出錯誤資料，會**污染整個 Event Bus**，讓所有消費者都收到錯誤資料。強制型別檢查在此尤其重要。

### JSON.stringify 容易被遺漏

在 `publish` 方法中直接對 `data` 呼叫 `JSON.stringify`，這是容易被忽略的細節。NATS client 的 `publish` 方法第二個參數**必須是字串或 Buffer**，如果直接傳入 Object，訊息會變成 `[object Object]`。

---

## 💡 重點摘要

- **Publisher 與 Listener 採用完全對稱的 Generic 設計**，Subject 與資料結構的約束機制相同。
- **Publisher 的錯誤更具破壞力**——一條錯誤的 Event 會影響所有消費者，型別檢查因此更關鍵。
- **`JSON.stringify` 是容易被遺漏的步驟**，BasePublisher 將其封裝在 `publish` 方法中，避免每次呼叫都要記得轉換。

---

## 🔑 關鍵字

Publisher, TicketCreatedPublisher, Generic, JSON.stringify, Event Bus
