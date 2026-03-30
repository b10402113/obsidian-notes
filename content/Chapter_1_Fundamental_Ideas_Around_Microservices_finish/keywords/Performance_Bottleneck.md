# Performance Bottleneck

## 定義

Performance Bottleneck（效能瓶頸）指的是在 service 通訊鏈中，整體回應速度永遠受限於最慢的那個環節。在 [[Synchronous Communication]] 中，若其中一個 service 的回應時間特別長，會拖累整個請求鏈。

## 範例

- 向 Service A 請求：10ms
- 向 Service C 請求：10ms
- 向 Service B 請求：**20 秒**

→ 整體處理時間 = **20 秒 + 20ms**（受限於最慢的 Service B）

## 相關概念

- [[Synchronous Communication]]
- [[Web of Dependencies]]
- [[Fault Propagation]]

## 來源

- [[5_Sync_Communication_Between_Services]]
