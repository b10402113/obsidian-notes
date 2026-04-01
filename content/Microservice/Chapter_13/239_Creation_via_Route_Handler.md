# Create Ticket 路由實作 — 建立並儲存票券

## 📝 課程概述

本節完成了 Create Ticket 路由的完整實作：在 Router handler 中呼叫 `Ticket.build()` 建立文件實體，使用 `await ticket.save()` 寫入 MongoDB，並回傳 HTTP 201 狀態碼。同時補充了測試案例，確保新建立的票券確實被寫入資料庫，並驗證資料內容的正確性。

## 核心觀念與實作解析

### 路由 Handler 的完整實作

```typescript
import { Ticket, build } from '../models/ticket';

router.post('/api/tickets', requireAuth, [...validators], validateRequest,
  async (req: Request, res: Response) => {
    const { title, price } = req.body;

    const ticket = Ticket.build({
      title,
      price,
      userId: req.currentUser!.id,  // 由 requireAuth 保证已定义
    });

    await ticket.save();

    res.status(201).send(ticket);
  }
);
```

### 為何 `userId` 來自 `req.currentUser`

`currentUser` middleware 會在 `req` 物件上附加登入使用者的資訊（id、email）。由於 `requireAuth` middleware 已經確保了 `currentUser` 必定存在才會進入 handler，所以這裡可以安心使用 `!` 斷言。這個設計讓每個建立的票券都自動帶有擁有者 ID，為未來的「只能編輯自己的票券」邏輯打下基礎。

### 回傳 201 而非 200

HTTP 201 Created 表示「請求成功，且伺服器因此建立了一個新資源」。使用 201 而非 200 是對 HTTP 語意的正確遵守，前端可以根據狀態碼判斷這是一次「新增」操作而非「讀取」操作。

### 測試驗證：確保資料真的寫入資料庫

```typescript
let tickets = await Ticket.find({});
expect(tickets.length).toEqual(0);

await request(app).post('/api/tickets').set('Cookie', global.signin()).send({ ... });

tickets = await Ticket.find({});
expect(tickets.length).toEqual(1);
expect(tickets[0].price).toEqual(20);
```

這個測試先確認資料庫一開始是空的，再建立票券，最後驗證數量與內容。不僅確認了 API 回傳了正確的 response，更確認了背後的資料持久化機制是正常運作的。

## 💡 重點摘要

- `Ticket.build()` + `await ticket.save()` 是建立並寫入票券的標準流程。
- `req.currentUser!.id` 之所以安全，是因為 `requireAuth` middleware 已經在進入 handler 前擋掉了未登入的請求。
- HTTP 201 是對「新資源建立成功」的正確語意回應。
- 測試不只要檢查 API 回應，還要直接查詢資料庫以驗證資料確實被持久化。

## 🔑 關鍵字

Ticket.build, ticket.save, req.currentUser, HTTP 201, mongoose find, requireAuth
