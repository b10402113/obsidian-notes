# Database per Service

## 定義

Database per Service 是 microservices 的核心設計原則：**每一個 service 在需要時，都會擁有自己的獨立 database，且 service 絕對不能直接讀取另一個 service 的 database**。

## 核心原因

1. **確保 service 的獨立性與高可用性**：一個資料庫故障不會拖垮所有 service
2. **避免 Schema Coupling**：一個 service 的 schema 變動不會影響其他 service
3. **允許選擇最適合的資料庫類型**：如 MongoDB 或 PostgreSQL

## 相關概念

- [[Schema Coupling]]
- [[Service Independence]]
- [[Fault Isolation]]
- [[Microservice]]

## 來源

- [[3_Data_in_Microservices]]
- [[5_Sync_Communication_Between_Services]]
- [[8_Pros_and_Cons_of_Async_Communication]]
