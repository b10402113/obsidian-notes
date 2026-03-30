# Service Decoupling

## 定義

Service Decoupling（服務解耦）指的是透過 [[Asynchronous Communication]] 或 [[Event-Driven Architecture]]，讓 service 之間不再有直接的呼叫依賴，進而提升系統的穩定性與可維護性。

## 與 Zero Dependency 的區別

- [[Service Decoupling]]：透過事件機制減少耦合
- [[Zero Dependency]]：Service D 完全不需要依賴其他 service（實現完全獨立）

## 相關概念

- [[Asynchronous Communication]]
- [[Event-Driven Architecture]]
- [[Event Bus]]

## 來源

- [[7_A_Crazy_Way_of_Storing_Data]]
