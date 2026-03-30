# Single Point of Failure

## 定義

Single Point of Failure（單一故障點）指的是系統中某個元件一旦故障，整個系統就會停止運作。[[Monolithic]] 架構中的共享資料庫是典型的單一故障點，而 [[Event Bus]] 在 Event-Driven Architecture 中也可能成為單一故障點。

## 對比

| 架構 | 故障點 |
|------|--------|
| Monolithic | 共享資料庫 |
| Microservices（第一種 Async） | Event Bus |

## 相關概念

- [[Monolithic]]
- [[Event Bus]]
- [[Fault Isolation]]
- [[Fault Tolerance]]

## 來源

- [[7_A_Crazy_Way_of_Storing_Data]]
