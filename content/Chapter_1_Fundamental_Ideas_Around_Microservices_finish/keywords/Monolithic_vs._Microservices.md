# Monolithic vs. Microservices

## 定義

Monolithic vs. Microservices 是兩種主流軟體架構模式的對比：

| 維度 | Monolithic | Microservices |
|------|-----------|---------------|
| 程式碼組織 | 所有功能在一個 codebase | 每個功能在獨立 service |
| 部署方式 | 單一部署單元 | 可獨立部署 |
| 資料庫 | 共享資料庫 | 每個 service 獨立資料庫 |
| 故障影響 | 單點故障 | 故障隔離 |
| 擴展方式 | 複製整個應用 | 只擴展需要更多資源的 service |

## 相關概念

- [[Monolithic]]
- [[Microservice]]
- [[Database per Service]]
- [[Fault Isolation]]

## 來源

- [[3_Data_in_Microservices]]
