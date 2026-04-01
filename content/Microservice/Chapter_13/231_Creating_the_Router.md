# 建立 Create Ticket 路由 — 測試驅動開發初探

## 📝 課程概述

本節進入 Tickets Service 的實作核心，首先以 **測試優先（Test-First）** 的方式規劃 Create Ticket 路由的測試案例，然後建立對應的 Router 檔案並在 `app.ts` 中註冊。本節的重點不在於馬上完成功能，而是學會用測試來定義「正確的行為」是什麼。

## 核心觀念與實作解析

### 測試驅動開發的意義

在開始寫實作之前，先把所有預期的行為寫成測試：

```typescript
// routes/__test__/new.test.ts
it('has a route handler listening on /api/tickets for POST requests', async () => {
  const response = await request(app)
    .post('/api/tickets')
    .send({});
  expect(response.status).not.toEqual(404);
});
```

為什麼先寫這個測試？如果 route 不存在，Express 會走到 catch-all 的 not-found handler，回傳 404。這個測試能確保 route handler 已經正確掛載。

### 路由的掛載與 Middleware 順序

在 `app.ts` 中：

```typescript
import { createTicketRouter } from './routes/new';
app.use(createTicketRouter);
```

掛載順序非常重要：需要認證的路由必須在 `app.use(currentUser)` 之後註冊，否則 `currentUser` middleware 無法正常運作。

### 初步 Router 的實作

```typescript
// routes/new.ts
import express, { Request, Response } from 'express';
const router = express.Router();

router.post('/api/tickets', (req: Request, res: Response) => {
  res.sendStatus(200);
});

export { router as createTicketRouter };
```

目前只是一個會回傳 200 的啞實作，目的是讓第一個測試儘快通過，確認整個流程（import → 掛載 → 收到 request）是通的。

## 💡 重點摘要

- **先寫測試，再寫實作**：用測試來定義預期行為，比寫完再測更有效率。
- Router 檔案的命名（`new.ts`）對應 Create 路由，符合 RESTful 命名慣例。
- `app.use()` 的順序很關鍵：Cookie Session → Current User → 路由，順序顛倒會導致認證失敗。
- 先用啞實作（啞資料回應）通過最基礎的路由存在性測試，再逐步加入完整邏輯。

## 🔑 關鍵字

Test-Driven Development, Express Router, Middleware, app.use, Jest
