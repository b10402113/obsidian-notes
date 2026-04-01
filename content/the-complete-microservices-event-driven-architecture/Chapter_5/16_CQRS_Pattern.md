# CQRS Pattern

## 📝 課程概述

本單元介紹 **CQRS（Command and Query Responsibility Segregation）Pattern**——一種將「寫入操作」與「讀取操作」在程式碼、資料模型、資料庫技術與部署策略上**完全分離**的架構模式。在 Microservice 與 Event-Driven Architecture 的加持下，CQRS 能同時解決「隔離關注點」與「跨服務資料JOIN」兩大難題，讓系統同時在讀寫兩條路徑上都達到最佳效能。

---

## 核心觀念與實作解析

### CQRS 的核心定義

在傳統系統中，幾乎所有對資料的操作都可以分為兩類：

- **Command（命令）**：導致資料變更的操作——新增（INSERT）、更新（UPDATE）、刪除（DELETE）。不會回傳資料本體，只回傳執行結果（成功/失敗）。
- **Query（查詢）**：純讀取資料的操作——SELECT、排序、聚合計算。**不會**改變資料庫內的資料。

CQRS 的核心思想就是：**把這兩種操作的路徑，從 Service 到 Database，徹底分開。**

```
[Command 路徑]
Client → Command Service → Command Database

[Query 路徑]
Client → Query Service → Query Database（包含 Command Database 的資料副本）
```

兩者之間同步資料的方式是：**Event-Driven——每次 Command Database 有變更，就發佈一個 Event 到 Message Broker，Query Service 監聽並更新自己的資料庫**。

---

### CQRS 的兩大應用場景

#### 場景一：隔離讀寫關注點，優化各自效能

當 Command 與 Query 分離成兩個獨立的 Service 之後，每個路徑都可以**根據自己的工作負擔特性，選用最適合的技術**。

**Command Service 的特性需求：**
- 嚴格的寫入權限管理
- 完整的 Input Validation
- 複雜的 Business Logic（例如：防機器人驗證、防止短時間重複評論）
- 需要對寫操作進行最佳化的資料庫 Schema 與技術

**Query Service 的特性需求：**
- 乾淨、簡單，專注於「如何最好地呈現資料給使用者」
- 對讀操作進行最佳化的資料結構（例如：預先計算好 Average Rating）
- 讀取密集（Read-intensive），需要不同的資料庫配置

> **為什麼這很重要？** 讀密集工作負擔與寫密集工作負擔的最佳化策略幾乎是對立的——用同一個資料庫同時滿足兩者，兩邊都無法達到最佳化。

**獨立擴展的好處：**
- 可以根據實際流量，分別調整 Command Service 與 Query Service 的 Instance 數量
- 也可以分別調整 Command Database 與 Query Database 的 Instance 數量
- 當只有 Query Service 需要 Scale 時，不需要 Scale Command Service——節省成本

**團隊協作的優勢：**
> 如果只有 Command Service 改動了 Business Logic 或 Validation，Query Service **完全不需要重新測試或重新部署**，因為 Query Service 的邏輯根本沒有變。

---

#### 場景二：解決跨 Microservice 資料 JOIN 的效能問題

這是 Microservice 架構下的一個經典困境。

在 Monolith 時代，所有資料在同一個 RDBMS 中，JOIN 是基本操作。但在 Microservice 架構下：

```
使用者請求：「找出評分最高的 sushi 餐廳」
→ 必須：
  1. 先呼叫 Business Service → 取得所有符合條件的商家
  2. 再呼叫 Review Service → 對每個商家取得評論與評分
  3. 在應用層手動計算平均分數並排序
```

這個**應用層 JOIN** 的代價極高——延遲嚴重，且當資料量大時幾乎不可接受。

**CQRS 的解法：建立一個專門的 Query Service，維護一份「JOIN View」**

```
[資料同步流程]
Business Service 變更資料 → 發佈 Event → Message Broker → Query Service 更新 JOIN View
Review Service 變更資料 → 發佈 Event → Message Broker → Query Service 更新 JOIN View

[讀取流程]
使用者查詢 → Query Service 直接回傳已預先 JOIN 好的資料
```

Query Service 只需要專心地「格式化並呈現資料」，Business Logic 完全不存在。

---

### CQRS + 餐廳評論平台的真實案例

**場景背景：** 一個群眾外包評論平台，使用者可以評論餐廳、美髮店等商家，其他使用者可以搜尋並依評分排序。

**Command 端的複雜性：**
- Business Service：需驗證是否為已認證的商家擁有者才能更新商家資訊
- Review Service：需防機器人驗證、確保使用者未在短期內重複評論、驗證評論內容格式等

**Query 端的複雜性：**
- 搜尋「sushi restaurant」，需要 JOIN Business 與 Review 兩份資料
- 需根據關鍵字相關性、平均評分、評論數量排序
- 這是一個高度讀密集的場景

**CQRS 解法：Business Search Service**

```
Command Side:
  Business Service（寫） ← Command Database
  Review Service（寫） ← Command Database
  ↓ 各自發佈事件
  Message Broker
  ↓ 監聽並消費
  Business Search Service（讀） ← 專門的 Text Search Engine Database（Elasticsearch 等）
```

Business Search Service 收到 Review 事件後，會自動重新計算該商家的平均評分。整個過程對外完全封裝，使用者只需向 Business Search Service 發送一個搜尋請求。

---

### CQRS 的關鍵限制：Eventual Consistency

使用 CQRS 後，Command Database 與 Query Database **不再是即時同步的**——資料從被寫入到 Query Database 更新之間，存在一個**時間窗口（Time Gap）**。

在這個時間窗口內，**某些讀操作會看到舊資料**。這就是 Eventual Consistency（最終一致性）。

> 這是一個有意義的取捨。如果你的業務場景**必須」看到剛寫入的資料**（例如：庫存系統不能超賣），CQRS 就不適合你。如果可以接受「稍晚一點看到新資料」（例如：搜尋排序稍微延遲），CQRS 就是極佳的選擇。

---

## 💡 重點摘要

- **CQRS 將 Command 與 Query 在 Service、Database 與部署策略上完全分離，讓讀寫兩條路徑都能獨立達到最佳化。**
- **Command Service 承載複雜的 Business Logic 與 Validation；Query Service 專注於呈現格式，兩者可以各自選擇最適合的資料庫技術。**
- **CQRS 解決了 Microservice 架構下「跨 Service JOIN」的根本問題——不再需要在應用層手動計算，而是由 Query Service 維護一份預先 JOIN 好的視圖。**
- **CQRS + Event-Driven Architecture 是業界常見的黃金組合：Command Service 發佈事件，Query Service 消費並同步更新。**
- **CQRS 只能保證 Eventual Consistency，而非強一致性——應用場景必須能接受這個 trade-off。**

---

## 🔑 關鍵字

CQRS, Command, Query, Command Service, Query Service, Eventual Consistency, Message Broker, Read-optimized Database, Write-optimized Database, Join View, Event-Driven Architecture
