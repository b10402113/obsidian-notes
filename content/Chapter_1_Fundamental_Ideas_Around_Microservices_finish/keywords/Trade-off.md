# Trade-off

## 定義

Trade-off（取捨）是系統設計中的核心思維——沒有完美的架構，只有適合當下需求的取捨。

## Data Replication 的 Trade-off

| 獲得 | 付出 |
|------|------|
| [[Zero Dependency]] | [[Data Duplication]] |
| 極致查詢效能 | 可能存在 [[Stale Data]] |

現代雲端的 [[Storage Cost]] 極低，使這個 trade-off 的代價大幅降低。

## 相關概念

- [[Data Replication]]
- [[Data Duplication]]
- [[Storage Cost]]
- [[Stale Data]]

## 來源

- [[8_Pros_and_Cons_of_Async_Communication]]
