# ProductCreated

## 定義

ProductCreated 是 [[Event Bus]] 中的一種 [[Event Type]]，由 Service B 在建立新商品時發出。事件包含新商品的 id、title、image 等資訊。

## 在 [[Data Replication]] 中的作用

Service D 監聽 ProductCreated 事件，收到後將商品的 id、title、image 存入自己的 Products Collection，實現跨 service 的資料同步。

## 相關概念

- [[Event Type]]
- [[Event Bus]]
- [[Data Replication]]
- [[UserCreated]]
- [[OrderCreated]]

## 來源

- [[8_Pros_and_Cons_of_Async_Communication]]
