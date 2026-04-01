# 建立 Custom Event Interface 與 TypeScript 泛型約束

## 📝 課程概述

本單元將 Subject 與 Event Data 的隱性耦合，轉化為顯式的 TypeScript Interface，並透過 **Generic（泛型）**讓 `Listener` 類別能自動推斷 `onMessage` 中 `data` 的型別。這是整個章節的核心技術——透過 TypeScript 的型別系統，在**編譯期就捕捉到 Subject 與資料結構的不匹配**，而不是等到訊息送達 Runtime 才發現。

---

## 核心觀念與實作解析

### 建立 TicketCreatedEvent Interface

```typescript
import { Subjects } from './subjects';

export interface TicketCreatedEvent {
  subject: Subjects.TicketCreated;
  data: {
    id: string;
    title: string;
    price: number;
  };
}
```

這裡的設計精神是：**用 Interface 明確綁定某一個 Subject 與其資料結構**。往後每一種 Event（`TicketUpdated`、`OrderCreated` 等）都會有一個對應的 Interface。

### Generic Class 的設計

```typescript
interface Event {
  subject: Subjects;
  data: any;
}

// T extends Event 表示：T 必須是一個 Event
export abstract class Listener<T extends Event> {
  abstract subject: T['subject'];  // Subject 型別自動與 Event 對齊
  abstract onMessage(data: T['data'], msg: Message): void;
}
```

### 為什麼 Subject 要用 `T['subject']` 而非直接寫 `Subjects`？

```typescript
// 如果直接寫 Subjects，子類可以任意賦值
subject = Subjects.OrderUpdated; // ❌ 危險！監聽錯誤的 Channel

// 如果寫成 T['subject']，TypeScript 會檢查必須與 Event 的 subject 完全一致
```

### 程式碼運作邏輯

當子類這樣定義時：

```typescript
class TicketCreatedListener extends Listener<TicketCreatedEvent> {
  subject: Subjects.TicketCreated = Subjects.TicketCreated; // ✅
  onMessage(data: TicketCreatedEvent['data'], msg: Message) {
    console.log(data.title); // ✅ TypeScript 知道是 string
    console.log(data.name);  // ❌ 編譯錯誤！name 不存在
  }
}
```

TypeScript 會自動：
1. **驗證 `subject` 必須是 `Subjects.TicketCreated`**（不能寫錯）
2. **驗證 `onMessage` 的 `data` 參數型別是 `{ id, title, price }`**

> **TypeScript Generic 的核心價值在於：它讓「Subject 與資料結構的對應關係」從「文件中提到的慣例」升級為「編譯器強制執行的規則」。**

### 為什麼需要 explicit type annotation？

```typescript
subject: Subjects.TicketCreated = Subjects.TicketCreated;
```

第二個 `Subjects.TicketCreated` 是**值**（runtime），第一個 `Subjects.TicketCreated` 是**型別 annotation**（compile-time）。兩者缺一不可——如果只給值，TypeScript 推斷出的型別會是 `Subjects`（整個 Enum），而非精確的 `Subjects.TicketCreated`，這會讓 Generic 約束失效。

---

## 💡 重點摘要

- **每一個 Subject 都應有對應的 Event Interface**，將隱性耦合顯式化。
- **`T extends Event` Generic 約束確保子類提供的 Subject 與 `onMessage` 的 `data` 型別自動對齊**。
- **`subject` 需要 explicit type annotation**，否則 TypeScript 無法執行精確約束。
- **這套機制的最大價值：Runtime 才會暴露的錯誤，提前到編譯期就攔截。**

---

## 🔑 關鍵字

TicketCreatedEvent, Generic, T extends Event, Subjects, Type Annotation
