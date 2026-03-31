# Eventual Consistency

最終一致性。相對於「強一致性」，非同步系統中其他 Service 的資料更新了，本地副本可能一時之間是舊的，但最終會達成一致。這是 [[Event Sourcing]] 模式需要處理的代價，也是 [[Zero Dependency]] 架構的交換條件。

## Reference

[[8_非同步通訊的優勢與資料複製策略.md]]
