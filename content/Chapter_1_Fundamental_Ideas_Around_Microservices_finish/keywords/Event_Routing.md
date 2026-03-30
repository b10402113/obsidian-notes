# Event Routing

## 定義

Event Routing（事件路由）是 [[Event Bus]] 的核心職能：當某個 service 向 Event Bus 發送事件時，Event Bus 會自動將該事件的副本路由（route）給所有對此事件感興趣的 service。

## 運作流程

1. Service 發送事件到 Event Bus（如 `UserQuery`）
2. Event Bus 識別對此事件感興趣的 service
3. 將事件副本分發給所有訂閱的 service

## 相關概念

- [[Event Bus]]
- [[Event Type]]
- [[Event-Driven Architecture]]

## 來源

- [[7_A_Crazy_Way_of_Storing_Data]]
