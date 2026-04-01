# TypeScript 驗證 Listener 的挑戰

## 📝 課程概述

本單元揭示了微服務中一個非常實際的問題：**Event 的 Subject 名稱（如 `ticket:created`）和 Event 攜帶的資料結構，是兩個高度相關但未被強制約束的資訊**。如果放任工程師自由輸入，容易產生「Subject 寫對了，但資料屬性卻拼錯了」的疏漏。TypeScript 的任務，就是把這層隱性關係**顯式化並加以強制**。

---

## 核心觀念與實作解析

### 問題的具體呈現

在 `onMessage` 中，工程師可能會這樣寫：

```typescript
onMessage(data: any, msg: Message) {
  console.log(data.name);    // ❌ 錯！Ticket 的屬性是 title，不是 name
  console.log(data.cost);    // ❌ 錯！Ticket 的屬性是 price，不是 cost
  msg.ack();
}
```

但 `data` 的型別是 `any`，TypeScript 不會報告任何錯誤。類似的問題還有 Subject 本身：

```typescript
subject = 'Ticket:Created'; // ❌ 大小寫錯誤，NATS 視為不同 Channel
```

### 第一步：建立 Subjects Enum

將所有合法的 Subject 名稱集中管理杜絕拼寫錯誤：

```typescript
export enum Subjects {
  TicketCreated = 'ticket:created',
  OrderUpdated = 'order:updated'
}
```

> **好處：Subject 名稱只在一處定義（Enum），所有 Service 透過 `Subjects.TicketCreated` 引用，大幅降低拼寫失誤的機會。**

### 為什麼 Enum 比 Const String 更安全？

如果你用 `const` 定義 Subject，工程師依然可能寫成：

```typescript
const subject = 'ticket:created'; // OK
const subject2 = 'tickt:created'; // 型別仍是 string，TS 不會攔截
```

但使用 Enum 時，`subject` 的型別是 `Subjects`，**TypeScript 會強迫你只能使用 Enum 中定義的值**。

### Subject 與 Event Data 的隱性耦合

| Subject | 預期資料結構 |
|---|---|
| `ticket:created` | `{ id: string, title: string, price: number }` |
| `order:updated` | `{ id: string, userId: string, ticketId: string }` |

每一個 Subject 都對應一個**唯一的資料結構**。這個對應關係目前沒有被 TypeScript 強制執行——接下來的單元將逐步實現這個約束。

---

## 💡 重點摘要

- **TypeScript 的 `any` 型別讓錯誤的屬性名稱完全暢通無阻**，這是大型微服務專案中的隱形風險。
- **Subjects Enum 將 Subject 名稱集中管理**，讓拼寫錯誤在編譯期就被攔截，而非等到 Runtime。
- **Subject 與 Event Data 之間存在一對一的隱性耦合**，這個關係需要被顯式化。

---

## 🔑 關鍵字

Subjects, Enum, Event Schema, ticket:created, Type Safety
