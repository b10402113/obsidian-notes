# Synchronous Communication

## 定義

Synchronous Communication（同步通訊）指的是：一個 service 透過直接發送 Request 的方式，向另一個 service 取得資料，是 **request → response** 的雙向溝通模式。

> 重要：此術語與 JavaScript 中的 async/await 概念完全不同，請勿混淆。

## 運作範例

Service D 向 Service A、B、C 依序發送 request，取得使用者資料、訂單資料與商品資料後，才能回傳最終結果。

## 優點

- 易於理解
- Service D 不需要有自己的資料庫

## 缺點

- 引入 [[Service Dependency]]
- 形成 [[Web of Dependencies]]
- 產生 [[Performance Bottleneck]]
- 導致 [[Fault Propagation]]

## 相關概念

- [[Request-Response Pattern]]
- [[Asynchronous Communication]]
- [[Database per Service]]

## 來源

- [[5_Sync_Communication_Between_Services]]
