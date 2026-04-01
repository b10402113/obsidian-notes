# 強制執行資料類型：完成 Listener 驗證

## 📝 課程概述

本單元完成了 Listener 端型別安全的最後一塊拼圖：將 `onMessage` 的 `data` 參數從 `any` 替換為 `T['data']`，讓 TypeScript 能夠**自動阻止我們存取不存在於 Event 結構中的屬性**。此時，我們建立了一個完整的「編譯期防火牆」——Subject 寫錯會被檢查，資料屬性拼錯也會被檢查。

---

## 核心觀念與實作解析

### 完成 BaseListener 的 onMessage 簽名

```typescript
abstract onMessage(data: T['data'], msg: Message): void;
```

此時 `T['data']` 會根據子類傳入的 Generic Type 自動解析。例如傳入 `TicketCreatedEvent`，則 `T['data']` 等於 `{ id: string; title: string; price: number }`。

### 子類的 onMessage 實作

```typescript
class TicketCreatedListener extends Listener<TicketCreatedEvent> {
  subject = Subjects.TicketCreated;

  onMessage(data: TicketCreatedEvent['data'], msg: Message) {
    // ✅ 自動補全：id, title, price
    console.log(data.id);
    console.log(data.title);
    console.log(data.price);

    // ❌ 編譯錯誤！TS 知道這些屬性不存在
    // console.log(data.name);
    // console.log(data.cost);

    msg.ack();
  }
}
```

### 防止錯誤的 Event Interface 應用

如果工程師不小心用了錯誤的 Event Interface：

```typescript
interface FakeData {
  name: string;
  cost: number;
}

// ❌ 這會直接觸發 TypeScript 編譯錯誤
class BadListener extends Listener<FakeData> { ... }
```

原因是 `FakeData` 不符合 `Event` 介面（沒有 `subject` 屬性），`T extends Event` 的約束會直接阻擋這個錯誤用法。

### TypeScript Generic 在微服務中的核心價值

| 問題 | 沒有 Generic | 有 Generic |
|---|---|---|
| Subject 拼錯 | Runtime 才能發現 | 編譯期檢測 |
| 屬性名稱拼錯 | `any` 不會攔截 | `T['data']` 阻止 |
| Event Type 用錯 | 不會報錯 | 直接編譯錯誤 |

> **在微服務的語境下，Team Member 之間對 Event 結構的認知不一致，是最常見的 Bug 來源之一。TypeScript Generic 將這個風險從「文件約定」轉變為「編譯器强制」。**

---

## 💡 重點摘要

- **`T['data']` 讓 `onMessage` 的資料參數具有精確的強型別**，存取不存在屬性會直接編譯失敗。
- **Generic 泛型不僅檢查 Subject，也檢查資料結構的完整性**——兩層把關，滴水不漏。
- **`FakeData` 這類錯誤的 Event Interface 會被 `T extends Event` 直接阻擋**，連 Runtime 的機會都沒有。

---

## 🔑 關鍵字

T['data'], Event Interface, Type Safety, Generic Constraint, ticket:created
