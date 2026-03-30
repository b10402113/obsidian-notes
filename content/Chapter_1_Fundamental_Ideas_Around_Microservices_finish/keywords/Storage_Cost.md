# Storage Cost

## 定義

Storage Cost（儲存成本）是評估 [[Data Duplication]] 時需要重新思考的維度。在現代雲端時代，儲存成本已經低到可以忽略不計：

- AWS / Google Cloud / Azure MySQL：約 **$0.015 ~ $0.0175 / GB / 月**
- 一筆商品資訊 API 回應：約 **1,200 bytes**
- 儲存 **1 億筆記錄**：月費約 **$14 美元**

## 結論

在雲端時代，用少許額外的儲存成本換取系統的穩定性與效能，是完全值得的 [[Trade-off]]。

## 相關概念

- [[Data Duplication]]
- [[Data Replication]]

## 來源

- [[8_Pros_and_Cons_of_Async_Communication]]
