# Show Route — 根據 ID 查詢單一票券

## 📝 課程概述

本節實作了 Show Route：接收 `GET /api/tickets/:id` 請求，透過 Mongoose 的 `Ticket.findById()` 查詢資料庫，找到時回傳 200 與票券資料，找不到時拋出 `NotFoundError`（由 error handler 轉為 404）。本節的重點在於實作 RESTful 路由的「資源不存在」處理邏輯，以及透過測試確保路由行為正確。

## 核心觀念與實作解析

### Router 的建立

```typescript
// routes/show.ts
import { NotFoundError } from '@zhx-tickets/common';

router.get('/api/tickets/:id', async (req: Request, res: Response) => {
  const ticket = await Ticket.findById(req.params.id);

  if (!ticket) {
    throw new NotFoundError();
  }

  res.send(ticket);
});
```

### `Ticket.findById()` 的行為

- 找到文件 → 回傳 `TicketDoc` 實例
- **找不到** → 回傳 `null`（不是拋錯）

所以我們必須主動檢查 `if (!ticket)`，找到時才繼續處理，否則拋出 `NotFoundError`。這個 error 會被統一 error handler 攔截並轉為 404 響應。

### 測試的兩個情境

**情境一：找不到資源**

```typescript
const id = new mongoose.Types.ObjectId().toHexString();
const response = await request(app)
  .get(`/api/tickets/${id}`)
  .expect(404);
```

**情境二：找到資源（先生成票券，再取回）**

```typescript
const createResponse = await request(app)
  .post('/api/tickets')
  .set('Cookie', global.signin())
  .send({ title: 'concert', price: 20 })
  .expect(201);

const ticketId = createResponse.body.id;

const fetchResponse = await request(app)
  .get(`/api/tickets/${ticketId}`)
  .expect(200);

expect(fetchResponse.body.title).toEqual('concert');
```

### 生成有效 ObjectId 的技巧

在測試中，`/api/tickets/fake-id` 會讓 Mongoose 拋出一個 **CastError**（因為 `fake-id` 不是有效的 12 bytes / 24 hex 字元格式），這個錯誤不是 `NotFoundError`，所以被 error handler 轉為 400 而非 404。我們使用：

```typescript
import mongoose from 'mongoose';
const id = new mongoose.Types.ObjectId().toHexString();
```

這樣就能生成一個格式正確但資料庫中不存在的 ObjectId，用於測試「找不到」的場景。

## 💡 重點摘要

- `findById()` 找不到時回傳 `null`，不是拋錯——必須手動檢查。
- `NotFoundError` 是業務層的「找不到」信號，由 error handler 統一轉為 HTTP 404。
- 測試中使用 `new mongoose.Types.ObjectId().toHexString()` 生成有效但不存在於 DB 的 ID，避免因格式無效而收到錯誤類型的錯誤。
- Show Route **不需要**身份驗證——任何人都可以查看票券資訊。

## 🔑 關鍵字

Ticket.findById, NotFoundError, CastError, ObjectId, GET route, HTTP 404
