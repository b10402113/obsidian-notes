# Event Sourcing

事件溯源。本課程採用的非同步通訊模式：每個 Service 維護自己的 [[Database per Service]]，其他 Service 在發生業務事件時主動發射 [[Event]]，目標 Service 據此更新自己的資料，實現 [[Zero Dependency]] 與 [[Eventual Consistency]]。

## Reference

[[8_非同步通訊的優勢與資料複製策略.md]]
