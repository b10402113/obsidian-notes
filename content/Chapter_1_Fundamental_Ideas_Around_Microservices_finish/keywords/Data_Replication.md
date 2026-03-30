# Data Replication

## 定義

Data Replication（資料複製）是第二種 [[Asynchronous Communication]] 模式的核心：讓每個 service 維護一份「符合自己需求」的資料副本，然後透過 [[Event Bus]] 在背景持續同步這些資料。

## 運作流程

1. Service A 處理使用者註冊，將新使用者存入自己的資料庫，**同時**發出 [[UserCreated]] 事件
2. Event Bus 將事件路由給 Service D
3. Service D 收到事件後，將對應資料存入自己的資料庫

## 優點

- 實現 [[Zero Dependency]]
- 極致的查詢效能（本地資料，無網路延遲）

## 缺點

- [[Data Duplication]]（可用 [[Storage Cost]] 換取穩定性）
- 可能產生 [[Stale Data]]

## 相關概念

- [[Event Bus]]
- [[Asynchronous Communication]]
- [[Zero Dependency]]
- [[Service D]]

## 來源

- [[8_Pros_and_Cons_of_Async_Communication]]
