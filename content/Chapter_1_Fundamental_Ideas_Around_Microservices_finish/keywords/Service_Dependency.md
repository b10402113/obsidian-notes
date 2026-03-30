# Service Dependency

## 定義

Service Dependency（服務依賴）指的是當一個 service 的運作需要依賴另一個 service 的正常運作時所形成的耦合關係。[[Synchronous Communication]] 是造成 service dependency 的主要原因。

## 與 microservices 目標的衝突

Microservices 的目標是讓每個 service 100% 獨立運作，但 Synchronous Communication 要求 Service D 必須依賴 Service A、B、C 的回應才能運作——這與獨立性目標直接衝突。

## 相關概念

- [[Synchronous Communication]]
- [[Web of Dependencies]]
- [[Zero Dependency]]

## 來源

- [[5_Sync_Communication_Between_Services]]
