# OrderCreated

## 定義

OrderCreated 是 [[Event Bus]] 中的一種 [[Event Type]]，由 Service C 在使用者訂購商品時發出，事件包含 userId 與 productId。

## 在 [[Data Replication]] 中的作用

Service D 監聽 OrderCreated 事件，收到後找到對應的使用者記錄，將 `productIds` 陣列更新，加入新訂購的 productId。

## 為什麼可能產生 [[Stale Data]]

Event Bus 尚未將 OrderCreated 事件傳遞給 Service D 前，剛好有查詢請求進來，就會看到尚未更新的資料。

## 相關概念

- [[Event Type]]
- [[Event Bus]]
- [[Data Replication]]
- [[ProductCreated]]
- [[UserCreated]]
- [[Stale Data]]

## 來源

- [[8_Pros_and_Cons_of_Async_Communication]]
