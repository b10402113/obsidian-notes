# UserCreated

## 定義

UserCreated 是 [[Event Bus]] 中的一種 [[Event Type]]，由 Service A 在新使用者註冊時發出，事件包含新使用者的 id。

## 在 [[Data Replication]] 中的作用

Service D 監聽 UserCreated 事件，收到後在自己的 Users Collection 中建立一筆記錄，初始化 `productIds` 為空陣列。

## 相關概念

- [[Event Type]]
- [[Event Bus]]
- [[Data Replication]]
- [[ProductCreated]]
- [[OrderCreated]]

## 來源

- [[8_Pros_and_Cons_of_Async_Communication]]
