# Saga Pattern

## 📝 課程概述

本單元介紹 Event-Driven Microservice 架構下的第一個核心設計模式——Saga Pattern。當我們將 Monolith 拆分為多個各自擁有獨立 Database per Service 的 Microservice 後，傳統 RDBMS 的 **ACID Transaction** 已經不復存在。Saga Pattern 就是用來在分散式環境下，**重建跨服務交易的最終一致性（Eventual Consistency）**的業界標準解法。

---

## 核心觀念與實作解析

### 問題的根本：ACID Transaction 的消失

在 Monolith 時代，所有資料都在同一個 Database 中，任意數量的 Table 都可以在同一個 Transaction 內同時更新——要么全部成功，要么全部回滾（Rollback），外部觀察者永遠只看到一個原子的操作。

然而當我們遷移到 Microservice 架構後：

- 每個 Microservice 擁有自己的 Database
- 每一個業務操作（例如「下一筆訂單」）橫跨多個 Microservice
- **沒有一個統一的 Transaction 可以覆蓋所有這些資料庫**

這就是 Saga Pattern 要解決的核心問題：**如何在分散式環境下，執行跨多個 Microservice 與 Database 的交易，同時保證最終一致性**。

---

### Saga Pattern 的核心概念

Saga 的核心思路是：**將一個大交易拆解為一連串的「本地交易（Local Transaction）」，每個本地交易由各自所屬的 Microservice 獨立執行。**

```
[成功流程]
操作 A（Service A）→ 操作 B（Service B）→ 操作 C（Service C）→ 完成
```

如果中途任何一個操作失敗，Saga 會執行**補償操作（Compensating Operation）**，以**相反順序**將已執行的操作一一撤銷。

> 補償操作並非 Rollback——它是一種「做相反的事」來達到相同結果。例如：扣款 → 補償是「退款」；預訂機位 → 補償是「取消機位」。

---

### 實作方式一：Workflow Orchestration（工作流編排）

第一種實作方式是引入一個專門的 **Stateful Workflow Orchestration Service**，這個 Service 的唯一職責就是：按照預先定義的順序，依次呼叫各 Microservice 的 API，並在失敗時觸發補償操作。

#### Vacation Booking 案例

場景：一個度假套裝服務，包含「機票 + 飯店 + 租車」三個元件。**要嘛全部預訂成功，要嘛全部取消**——不允許出現「扣了款但飯店沒預訂成功」的中間狀態。

工作流定義（含補償操作）：

| 步驟 | 操作 | 補償操作 |
|---|---|---|
| 1 | 向使用者扣款 | 退還款項 |
| 2 | 預訂來回機票 | 取消機票 |
| 3 | 預訂飯店房間 | 取消飯店 |
| 4 | 預訂租車 | 取消租車 |
| 5 | 在 Order Service 建立訂單 | 將訂單標記為已取消 |

#### 失敗時的補償邏輯

假設在「租車」步驟失敗（該地點當天已無庫存）：

```
Workflow Orchestration Service 啟動補償鏈：
  1. 呼叫飯店 Service → 取消飯店預訂
  2. 呼叫機票 Service → 取消機票
  3. 呼叫支付 Service → 退還全額款項
  4. 回傳錯誤訊息給使用者
```

這個模式的好處是：**整個工作流的狀態集中管理，失敗邏輯一目了然**。缺點是 Orchestration Service 本身成為一個需要維護的 Stateful Service，必須確保其可靠性。

---

### 實作方式二：Event-Driven Choreography（事件驅動編舞）

第二種實作方式**完全移除 Orchestration Service**，將工作流的控制權下放給各個 Microservice 自己。通訊全部透過 Message Broker 以**非同步事件**進行。

每個 Microservice 必須知道兩件事：
- 當自己執行成功時，**該發佈什麼事件**給下一個 Service
- 當自己執行失敗時，**該發佈什麼事件**來觸發前一 Service 的補償邏輯

#### Vacation Booking 的事件鏈

```
[成功路徑]
Payment Service：扣款 → 發佈 PaymentCompleted 事件
  → Flight Service（監聽）收到：預訂機票 → 發佈 FlightBooked 事件
    → Hotel Service（監聽）收到：預訂飯店 → 發佈 HotelBooked 事件
      → Car Rental Service（監聽）收到：預訂租車 → 發佈 AllCompleted 事件
        → Order Service（監聽）收到：建立訂單

[失敗路徑：租車失敗]
Car Rental Service：發佈 CarReservationFailed 事件
  → Hotel Service（監聽）收到：取消飯店 → 發佈 HotelCancelled 事件
    → Flight Service（監聽）收到：取消機票 → 發佈 FlightCancelled 事件
      → Payment Service（監聽）收到：退還款項 → 發佈 TransactionRolledBack 事件
        → Notification Service（監聽）收到：通知使用者失敗原因與後續步驟
```

> **注意**：因為整個流程是事件驅動的，成功/失敗的通知也必須是非同步的——透過 Notification Service 以 Email 或 Push Notification 告知使用者。

---

### 兩種實作方式的比較

| 維度 | Orchestration | Event-Driven Choreography |
|---|---|---|
| 中心化程度 | 集中（Orchestrator 掌控全流程） | 分散（每個 Service 自管其職責） |
| 失敗邏輯可讀性 | 高（在一處定義） | 低（分散在各 Service 中） |
| 新增步驟的複雜度 | 需修改 Orchestrator | 需通知所有參與者 |
| 適用場景 | 步驟固定、流程複雜 | 步驟可能動態擴展 |
| 彈性 | 較低 | 較高 |

---

## 💡 重點摘要

- **Saga Pattern 是分散式交易（Distributed Transaction）的解決方案，用一系列本地交易加補償操作取代單一原子交易。**
- **補償操作不是 Rollback——它是一種「做相反的事」來達到撤銷效果，且是業務層面的撤銷而非資料庫層面的回滾。**
- **Workflow Orchestration 模式適合流程固定、步驟可預期的場景；Event-Driven Choreography 適合需要高度彈性與擴展性的場景。**
- **Saga 只能保證最終一致性（Eventual Consistency），而非 ACID 等級的強一致性。**

---

## 🔑 關鍵字

Saga Pattern, Compensating Operation, Workflow Orchestration, Event-Driven Choreography, Distributed Transaction, ACID, Eventual Consistency, Message Broker, Local Transaction
