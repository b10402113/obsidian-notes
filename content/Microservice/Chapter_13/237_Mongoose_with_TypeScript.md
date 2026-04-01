# Mongoose 與 TypeScript 的三元 Interface 模式

## 📝 課程概述

本節回顧了在 TypeScript 環境中使用 Mongoose 的標準模式：建立三個互補的 Interface（`TicketAttrs`、`TicketDoc`、`TicketModel`）來分別描述「建立記錄所需的資料」、「MongoDB 實際文件的型別」，以及「Model 本身的方法」）。這個模式解決了 TypeScript 對 Mongoose 動態屬性型別推斷不足的問題，讓我們在建立資料庫文件時也能享有完整的型別檢查。

## 核心觀念與實作解析

### 三個 Interface 的分工

| Interface | 用途 | 繼承 |
|-----------|------|------|
| `TicketAttrs` | 建立新文件時傳入的屬性（不含自動欄位） | 無 |
| `TicketDoc` | 已儲存文件的完整型別（含 Mongoose 自動附加的欄位） | `extends mongoose.Document` |
| `TicketModel` | Model 本身（含靜態方法） | `extends mongoose.Model<TicketDoc>` |

```typescript
interface TicketAttrs {
  title: string;
  price: number;
  userId: string;
}

interface TicketDoc extends mongoose.Document {
  title: string;
  price: number;
  userId: string;
}

interface TicketModel extends mongoose.Model<TicketDoc> {
  build(attrs: TicketAttrs): TicketDoc;
}
```

### 為何需要 `build()` 靜態方法

直接用 `new TicketModel({...})` 建立文件時，TypeScript **不會檢查**傳入的屬性是否完整（少了 `userId` 也不會报错）。`build()` 方法則會對照 `TicketAttrs` interface 進行型別檢查：

```typescript
ticketSchema.statics.build = (attrs: TicketAttrs) => {
  return new Ticket(attrs);
};
```

### `_id` 與 `id` 的轉換

MongoDB 儲存的 ID 欄位是 `_id`，但 API 回傳給前端時應該用更友善的 `id`（不含底線）。我們透過 Mongoose schema 的 `toJSON` transform 來自動處理：

```typescript
new mongoose.Schema({ ... }, {
  toJSON: {
    transform(doc, ret) {
      ret.id = ret._id;
      delete ret._id;
    },
  },
});
```

這個轉換的好處是：無論在程式哪個環節取用文件，JSON 輸出永遠是一致的 `id` 格式，方便前端處理。

### Schema 中 `type` 的大寫與小寫

在 Mongoose schema 中，`type: String`（大寫 S）指的是 JavaScript 的全域 `String` 建構函式，這是 Mongoose 自己用來判斷欄位型別的方式。**不要和 TypeScript 的 `type: string`（小寫 s）搞混**，後者是 TypeScript 的型別系統，兩者是完全不同的概念，只是剛好命名相似。

## 💡 重點摘要

- 三個 Interface 分工明確：`TicketAttrs`（建立時用）、`TicketDoc`（文件本身）、`TicketModel`（含 `build()` 静态方法）。
- `build()` 是確保 TypeScript 在建立文件時也能型別檢查的關鍵手段。
- Schema 的 `toJSON` transform 讓 `_id` → `id` 的轉換在所有輸出點自動生效。
- Schema 中的 `type: String`（大寫）與 TypeScript 的 `string`（小寫）是不同的系統，千萬不要混淆。

## 🔑 關鍵字

Mongoose, TypeScript Interface, Schema, build, toJSON, mongoose.Document
