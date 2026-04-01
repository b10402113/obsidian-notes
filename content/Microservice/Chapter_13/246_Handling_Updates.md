# Update Route — 編輯票券與擁有權驗證

## 📝 課程概述

本節實作了 Update Route（`PUT /api/tickets/:id`），這是 Tickets Service 中最複雜的路由，因為它涉及多層檢查：身份驗證（`requireAuth`）、資源存在性（`NotFoundError`），以及**擁有權驗證**（`NotAuthorizedError`）。更新時使用 Mongoose 的 `ticket.set()` 在記憶體中修改文件，再用 `await ticket.save()` 寫回資料庫。

## 核心觀念與實作解析

### 路由的多層 Middleware

```typescript
router.put('/api/tickets/:id',
  requireAuth,                           // 1. 必須登入
  [body('title').notEmpty(), ...],        // 2. 驗證輸入
  validateRequest,                        // 3. 檢查驗證結果
  async (req, res) => {
    const ticket = await Ticket.findById(req.params.id);
    if (!ticket) throw new NotFoundError();

    // 4. 擁有權檢查：只有建立者才能修改
    if (ticket.userId !== req.currentUser!.id) {
      throw new NotAuthorizedError();
    }

    ticket.set({ title: req.body.title, price: req.body.price });
    await ticket.save();

    res.send(ticket);
  }
);
```

### 為何需要擁有權檢查

這是微服務架構中「資料完整性」的核心體現：票券屬於建立它的使用者，只有擁有者才能修改定價或標題。如果缺少這個檢查，任何登入的使用者都可以修改他人的票券資料，造成明顯的安全漏洞。

### `ticket.set()` 的使用方式

```typescript
ticket.set({ title: req.body.title, price: req.body.price });
await ticket.save();
```

`set()` 只會修改記憶體中的文件，不會直接寫入資料庫。呼叫 `save()` 後，Mongoose 才會真正將變更持久化，並觸發任何已定義的 pre-save / post-save hooks。這也是為何 `save()` 後 `ticket` 物件已經是更新後的狀態，不需要重新查詢。

### 測試中模擬「不同使用者」

由於 `signin()` 現在每次都會產生隨機 ObjectId，呼叫兩次 `signin()` 會得到兩個不同的使用者身份：

```typescript
const cookieOne = global.signin();  // 使用者 A
const cookieTwo = global.signin();  // 使用者 B（不同 ID）

// 使用者 A 建立票券
const { body: { id } } = await request(app)
  .post('/api/tickets')
  .set('Cookie', cookieOne)
  .send({ title: 'concert', price: 20 });

// 使用者 B 企圖修改 → 應收到 401
await request(app)
  .put(`/api/tickets/${id}`)
  .set('Cookie', cookieTwo)
  .send({ title: 'new title', price: 100 })
  .expect(401);
```

## 💡 重點摘要

- Update Route 需要三層檢查：身份驗證 → 資源存在性 → 擁有權。
- `ticket.set()` 修改記憶體中的文件，`ticket.save()` 才真正寫入資料庫。
- `signin()` 每次產生隨機 ObjectId，天然模擬了多使用者的情境。
- 擁有權檢查 (`NotAuthorizedError`) 是防止未授權修改的關鍵防線。

## 🔑 關鍵字

PUT route, NotAuthorizedError, ticket.set, ticket.save, requireAuth, 擁有權檢查
