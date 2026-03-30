# Asynchronous Communication

## 定義

Asynchronous Communication（非同步通訊）指的是「發送端發出請求後，不需要等待接收端立即回應」的通訊模式，與 JavaScript 中的 async/await 概念完全不同。

> ⚠️ 重要澄清：在 microservices 領域，Async 的意義是「發送端發出請求後，不需要等待接收端立即回應」，與 JavaScript 的非同步程式執行是完全不同的概念。

## 兩種 Async 模式

1. **第一種**：透過 [[Event Bus]] 即時路由事件，但 Service D 仍需要依賴其他 service 的回應（缺陷與同步通訊相同）
2. **第二種**：透過 [[Data Replication]]，讓 Service D 維護自己的資料副本，實現 [[Zero Dependency]]

## 相關概念

- [[Event Bus]]
- [[Event-Driven Architecture]]
- [[Synchronous Communication]]
- [[Data Replication]]

## 來源

- [[7_A_Crazy_Way_of_Storing_Data]]
