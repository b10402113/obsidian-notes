# Index Route — 查詢所有票券

## 📝 課程概述

本節實作了 Index Route：`GET /api/tickets` 會從 MongoDB 取出所有票券並回傳一個陣列。這個路由不需要身份驗證，任何人都能查看票券列表。同時介紹了在測試中用 Helper Function 封裝重複的票券建立邏輯，讓測試程式碼更簡潔。

## 核心觀念與實作解析

### Index Route 的實作

```typescript
// routes/index.ts
router.get('/api/tickets', async (req: Request, res: Response) => {
  const tickets = await Ticket.find({});  // 空物件 = 不加任何過濾條件
  res.send(tickets);
});
```

`Ticket.find({})` 傳入一個空物件表示「不做任何過濾，取出集合中的所有文件」，回傳結果是一個陣列。

### 為何不需要身份驗證

票券列表是公開資訊，任何人都可以瀏覽有哪些票正在販售。只有**建立**與**編輯**票券才需要登入。

### Helper Function 避免重複

在測試中建立票券是常見操作，如果每次都寫這段會很冗長：

```typescript
const createTicket = () => {
  return request(app)
    .post('/api/tickets')
    .set('Cookie', global.signin())
    .send({ title: 'concert', price: 20 });
};

await createTicket();
await createTicket();
await createTicket();
```

將建立流程封裝成一個函式，讓測試讀起來更清晰，也便於日後修改建立邏輯。

### 測試驗證

```typescript
await createTicket();
await createTicket();
await createTicket();

const response = await request(app)
  .get('/api/tickets')
  .expect(200);

expect(response.body.length).toEqual(3);
```

直接檢查回傳陣列的長度，確保所有票券都被正確取出。

## 💡 重點摘要

- `Ticket.find({})` 的空物件參數表示「無過濾，取全部」。
- Index Route 應該對外公開，不需要 requireAuth middleware。
- Helper Function 是減少測試中重複程式碼的有效手段。
- 直接檢查回傳陣列的 `length` 是驗證「列表功能」是否正常的最直接方式。

## 🔑 關鍵字

Ticket.find, GET route, Index, Helper Function, Test, response.body
