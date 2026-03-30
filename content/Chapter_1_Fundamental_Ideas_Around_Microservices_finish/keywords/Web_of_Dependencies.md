# Web of Dependencies

## 定義

Web of Dependencies（依賴網絡）指的是在 Synchronous Communication 架構中，表面上只有幾個 service 的呼叫，但每個 service 的內部實作可能又呼叫了更多 service，形成複雜的深層依賴鏈。

## 風險

- 隱藏在表面呼叫背後的深層呼叫難以預測
- 鏈中任何一個節點失敗，整個請求就失敗
- 速度永遠受限於最慢的那一環

## 相關概念

- [[Service Dependency]]
- [[Synchronous Communication]]
- [[Performance Bottleneck]]

## 來源

- [[5_Sync_Communication_Between_Services]]
