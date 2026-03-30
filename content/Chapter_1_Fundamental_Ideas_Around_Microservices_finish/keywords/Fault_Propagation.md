# Fault Propagation

## 定義

Fault Propagation（故障傳播）指的是當通訊鏈中某個 service 故障時，該故障沿著依賴關係擴散，導致原本運作正常的 service 也跟著失敗的現象。這是 [[Synchronous Communication]] 的主要缺點之一。

## 影響

- Service A 故障 → Service D 也無法運作
- 任何一個子請求失敗，整體請求就跟著失敗
- 與 microservices 的 [[Fault Isolation]] 目標直接衝突

## 相關概念

- [[Synchronous Communication]]
- [[Service Dependency]]
- [[Fault Isolation]]

## 來源

- [[5_Sync_Communication_Between_Services]]
