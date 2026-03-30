# Service D

## 定義

Service D 是課程案例中的第四個 service，功能是「顯示某位使用者曾訂購的所有商品（含標題與圖片）」。它是展示 [[Synchronous Communication]] 缺陷與第二種 [[Asynchronous Communication]] 模式優勢的關鍵角色。

## 在 Synchronous Communication 中的角色

Service D 需要依賴 Service A（使用者資料）、Service B（商品資料）、Service C（訂單資料），形成強耦合。

## 在第二種 Async 模式中的角色

Service D 透過 [[Data Replication]] 維護自己的 Users Collection（含 `productIds`）與 Products Collection，實現 [[Zero Dependency]]——完全不需要向外呼叫任何其他 service 就能即時回應查詢。

## 相關概念

- [[Zero Dependency]]
- [[Data Replication]]
- [[Asynchronous Communication]]
- [[Synchronous Communication]]

## 來源

- [[8_Pros_and_Cons_of_Async_Communication]]
