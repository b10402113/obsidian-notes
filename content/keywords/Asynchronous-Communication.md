# Asynchronous Communication

非同步通訊。[[Microservice Communication]] 的另一大類，Service 不需要同時在線即可溝通。本課程採用的是 [[Event Sourcing]] 模式：每個 Service 維護自己的 Database，透過事件持續同步所需資料，實現 [[Zero Dependency]]。

## Reference

[[7_事件匯流排的非同步通訊方式.md]] | [[8_非同步通訊的優勢與資料複製策略.md]]
