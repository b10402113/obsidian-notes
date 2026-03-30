# Zero Dependency

## 定義

Zero Dependency（零依賴）是第二種 [[Asynchronous Communication]] 模式的核心目標：透過 [[Data Replication]]，讓 Service D 完全不需要向外呼叫其他 service，即使 Service A、B、C 全部掛掉，Service D 仍可正常回應查詢請求。

## 實現方式

Service D 維護一個包含用戶 ID、已購商品 ID 的本地資料庫，所有資料透過 [[Event Bus]] 同步自其他 service。

## 相關概念

- [[Service D]]
- [[Data Replication]]
- [[Asynchronous Communication]]
- [[Service Dependency]]

## 來源

- [[8_Pros_and_Cons_of_Async_Communication]]
