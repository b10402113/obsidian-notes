# 完整更新流程測試與 Ingress 配置

## 📝 課程概述

本節補完了 Update Route 的最後兩個測試（輸入驗證、快樂路徑），並透過 Postman 對所有四條路由進行了端到端的手動驗證。同時更新了 Kubernetes Ingress 配置，讓外部流量能夠正確路由到 Tickets Service。

## 核心觀念與實作解析

### 更新路由的完整測試

**驗證失敗的案例（空標題、負數價格）：**

```typescript
// 無效標題
await request(app)
  .put(`/api/tickets/${id}`)
  .set('Cookie', cookie)
  .send({ title: '', price: 20 })
  .expect(400);

// 無效價格（負數）
await request(app)
  .put(`/api/tickets/${id}`)
  .set('Cookie', cookie)
  .send({ title: 'valid title', price: -10 })
  .expect(400);
```

**快樂路徑（成功更新並驗證結果）：**

```typescript
const updateResponse = await request(app)
  .put(`/api/tickets/${id}`)
  .set('Cookie', cookie)
  .send({ title: 'new concert', price: 100 })
  .expect(200);

// 再次取回票券，確認資料已更新
const fetchResponse = await request(app)
  .get(`/api/tickets/${id}`)
  .expect(200);

expect(fetchResponse.body.title).toEqual('new concert');
expect(fetchResponse.body.price).toEqual(100);
```

### Ingress 配置的路由順序

```yaml
# ingress-srv.yaml
- path: /api/tickets/?(.*)
  pathType: Prefix
  backend:
    service:
      name: tickets-srv
      port:
        number: 3000
```

**路由規則的順序非常關鍵。** Catch-all（`path: /api/*`）規則必須放在最後，否則它會優先匹配所有請求，導致 `/api/tickets/*` 和 `/api/users/*` 永遠無法被正確路由。

### Postman 手動測試流程

使用 Postman 驗證完整流程：

1. `POST /api/users/signup` → 取得 Cookie
2. `POST /api/tickets`（帶 Cookie）→ 建立票券，獲得 `ticket.id`
3. `GET /api/tickets/:id` → 確認票券存在
4. `GET /api/tickets` → 確認列表包含新票券
5. `PUT /api/tickets/:id` → 更新票券
6. 再次 `GET /api/tickets/:id` → 確認更新已持久化

### 為何需要 Ingress 配置

Kubernetes 的 Ingress 是叢集對外的流量入口。沒有正確的 Ingress 路由規則，從外部發送到 `/api/tickets/*` 的請求根本無法被轉發到 `tickets-srv`，會被直接拒絕或轉到錯誤的 Service。

## 💡 重點摘要

- Update Route 的驗證規則（`notEmpty`、`isFloat({ gt: 0 })`）與 Create Route 完全一致。
- 快樂路徑測試中，取回更新後的票券驗證資料正確性，是確認 `save()` 確實生效的最嚴謹方式。
- Ingress YAML 中的路由規則順序是成敗關鍵：具體路徑必須寫在 catch-all 之前。
- Postman 手動測試能發現單元測試難以捕捉的問題（如 Ingress 配置錯誤），是 CI 流程外的重要驗證手段。

## 🔑 關鍵字

Ingress, Postman, End-to-End Test, catch-all route, ticket.save, 端到端驗證
