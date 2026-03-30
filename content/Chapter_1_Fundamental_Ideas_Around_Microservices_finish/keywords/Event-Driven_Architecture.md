# Event-Driven Architecture

## 定義

Event-Driven Architecture（事件驅動架構）是一種以事件的發送與接收為核心的通訊模式。當某個 service 發生資料變更時，會主動發出事件，其他感興趣的 service 負責接收並更新自己的資料庫。

## 核心元件

- **[[Event Bus]]**：負責路由事件
- **[[Event Type]]**：描述事件種類
- **[[Event Routing]]**：將事件副本分發給感興趣的 service

## 優點

- 實現 [[Service Decoupling]]
- 支援 [[Data Replication]]

## 缺點

- [[Single Point of Failure]]（Event Bus 本身）
- 概念複雜度較高

## 相關概念

- [[Event Bus]]
- [[Asynchronous Communication]]
- [[Data Replication]]

## 來源

- [[7_A_Crazy_Way_of_Storing_Data]]
- [[8_Pros_and_Cons_of_Async_Communication]]
