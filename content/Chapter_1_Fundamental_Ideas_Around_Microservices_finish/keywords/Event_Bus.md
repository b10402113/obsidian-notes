# Event Bus

## 定義

Event Bus（事件匯流排）是所有 service 共享的通訊中樞，專門用來處理各 service 發出的通知與事件。當某個 service 向 Event Bus 發送事件時，Event Bus 會自動將該事件的副本路由（route）給所有對此事件感興趣的 service。

## 事件結構

每個事件由以下部分組成：
- **[[Event Type]]**：描述事件的種類（如 `UserQuery`、`ProductCreated`）
- **data**：與事件相關的上下文資訊（如目標使用者的 ID）

## ⚠️ 單點故障風險

所有 service 都連接到同一個 Event Bus，因此 Event Bus 本身成為了 [[Single Point of Failure]]，必須額外做好高可用性設計。

## 相關概念

- [[Event-Driven Architecture]]
- [[Event Routing]]
- [[Asynchronous Communication]]
- [[Single Point of Failure]]
- [[Service D]]

## 來源

- [[7_A_Crazy_Way_of_Storing_Data]]
- [[8_Pros_and_Cons_of_Async_Communication]]
