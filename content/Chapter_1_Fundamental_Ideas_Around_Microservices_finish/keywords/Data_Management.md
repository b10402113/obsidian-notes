# Data Management

## 定義

Data Management（資料管理）是 microservices 架構中最大也最關鍵的挑戰——當每個 service 都有自己的資料庫，且不准直接讀取其他 service 的資料庫時，如何在 service 之間共享與管理資料。

## 兩種主要解決策略

1. **Synchronous Communication**：Service D 即時向其他 service 請求資料
2. **Asynchronous Communication + Data Replication**：Service D 維護自己的資料副本，透過 [[Event Bus]] 同步

## 相關概念

- [[Database per Service]]
- [[Synchronous Communication]]
- [[Asynchronous Communication]]
- [[Data Replication]]

## 來源

- [[3_Data_in_Microservices]]
