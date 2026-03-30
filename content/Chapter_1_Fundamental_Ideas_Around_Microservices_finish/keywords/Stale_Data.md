# Stale Data

## 定義

Stale Data（過時資料）是 [[Data Replication]] 模式的代價之一：由於 Service D 的資料是透過事件非同步填充的，在極少數情況下可能出現短暫的資料延遲——例如 Service C 發出 [[OrderCreated]] 事件後，Event Bus 尚未將事件傳遞給 Service D 前，剛好有查詢請求進來。

## 接受度

這是此模式需要接受的一個 [[Trade-off]]，但換來的是效能與穩定性的巨大提升。

## 相關概念

- [[Data Replication]]
- [[Trade-off]]
- [[Event Bus]]

## 來源

- [[8_Pros_and_Cons_of_Async_Communication]]
