# Data Duplication

## 定義

Data Duplication（資料重複）是 [[Data Replication]] 模式下的必然代價——每個 service 都會儲存一份符合自身需求的資料副本。這個缺點常被過度放大，但現代雲端儲存成本極低（AWS MySQL 約 $0.015/GB），**儲存 1 億筆記錄月費僅約 $14 美元**。

## 評估

用少許額外的儲存成本換取：
- 系統整體的穩定性
- 極致的查詢效能
- Service 的 [[Zero Dependency]]

## 相關概念

- [[Data Replication]]
- [[Storage Cost]]
- [[Trade-off]]

## 來源

- [[8_Pros_and_Cons_of_Async_Communication]]
