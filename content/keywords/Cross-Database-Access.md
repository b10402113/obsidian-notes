# Cross-Database Access

禁止直接跨 Service 讀取另一個 Service 的 Database。違反此規則會引入不必要的耦合，讓一個 Service 的故障影響到另一個 Service，是 Microservice 架構中的鐵律。

## Reference

[[3_微服務中的資料管理挑戰.md]]
