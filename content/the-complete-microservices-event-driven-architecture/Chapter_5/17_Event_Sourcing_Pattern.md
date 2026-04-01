# Event Sourcing Pattern

## 📝 課程概述

本單元介紹 **Event Sourcing Pattern**——一種與傳統「儲存當前狀態」完全不同的資料建模方式。傳統 CRUD 模式下，資料庫只保存「最新狀態」，歷史變動被覆蓋消失。而 Event Sourcing 的核心思維是：**不要只存結果，要把「所有發生過的事」全部記錄下來**。這種模式在金融、庫存、審計等需要完整歷史軌跡的場景中，有著無可取代的價值。

---

## 核心觀念與實作解析

### 傳統 CRUD 模式：只存「現在的狀態」

在大多數應用程式中，資料庫的內容反映的是**當前的業務狀態**：

- 使用者更新了商品價格 → 舊價格**被直接覆蓋**
- 銀行客戶餘額變動 → 只保存**最新餘額**，交易明細通常是另一張表的附加資訊

這種模式在多數場景下完全足夠——使用者只在乎**「現在是多少」**，不在乎「怎麼變成這樣的」。

**但有些場景，這種模式是致命的。**

---

### 什麼情境需要「知道如何抵達現狀」？

**銀行帳戶案例：**

> 如果銀行只給你「目前餘額 10,000 元」，卻不告訴你存款明細、提款記錄、費用扣除與轉帳紀錄——這是完全無法接受的。

餘額的**計算過程**本身就具有價值：不只是視覺化，更用於**稽核（Auditing）**與**潛在的修正**。

**庫存管理案例：**

> 早上登入系統，商品 A 顯示庫存 200 件；晚上再登入，變成 100 件。

只看到「100 件」這個結果，你**完全無法判斷**：
- 是 100 位顧客各買了 1 件？
- 是賣了 200 件然後退了 100 件？
- 還是一次性被某公司大額採購了？

> **「只看結果」是不夠的——我需要看到「通往結果的每一個步驟」。**

---

### Event Sourcing 的核心概念

Event Sourcing 的建模方式與傳統 CRUD 截然不同：

| 傳統 CRUD | Event Sourcing |
|---|---|
| 儲存**當前狀態**（State） | 儲存**所有事件**（Events） |
| UPDATE 直接覆蓋舊值 | **APPEND** 新事件，歷史永不消失 |
| 事件記錄是附屬品 | **事件本身就是主要資料** |

> **Events 是 Immutable（不可變）的**——一旦寫入系統，絕對不會被修改或刪除。我們唯一能做的事，是在日誌末尾**追加新事件**。

**銀行帳戶的 Event Sourcing 版本：**

```
[Event Log]
Deposit: $1,000    (帳戶開戶存入)
Withdraw: $200      (提款)
Deposit: $500       (轉帳收入)
Fee: $5             (手續費)
Withdraw: $100      (提款)

[Current Balance = $1,195]
透過 replay 整份事件日誌：1000 - 200 + 500 - 5 - 100 = $1,195
```

每次想知道「目前餘額」時，**重播（Replay）帳戶創建以來的所有事件**即可。

---

### Event Sourcing 的額外價值：從歷史資料中挖掘洞見

儲存完整事件日誌，不只是為了能重建狀態——更是為了**對歷史資料進行分析與監控**：

- **發現重複交易**：可能存在意外重複扣款的問題
- **偵測可疑行為**：可識別出不正常的交易模式，防止詐騙
- **提供理財建議**：根據消費習慣建議儲蓄帳戶或信用卡

這些分析在傳統 CRUD 模式下幾乎不可能做到，因為歷史資料早已被覆蓋了。

---

### 儲存事件的方式：Database vs. Message Broker

#### 方式一：以 Database 儲存事件（Append-Only Table）

以電子商務的訂單為例：

```
[Orders Event Log Table]
order_id | event_type          | timestamp
ORD-001  | OrderPlaced         | 2026-04-01 10:00
ORD-001  | PaymentConfirmed    | 2026-04-01 10:01
ORD-001  | Shipped             | 2026-04-01 14:00
ORD-001  | Delivered           | 2026-04-01 18:00
```

> 每一個狀態變更，都附加一筆新紀錄。

**這種方式適合：**
- 需要對整個資料集進行複雜查詢與分析（Analytics）
- 例如：分析某一天所有訂單的狀態轉換分佈，找出配送問題的高峰時段

#### 方式二：以 Message Broker 儲存事件

不同於存放在單一 Service 的私有資料庫，**將事件發佈到 Message Broker**，讓所有有興趣的 Consumer 都可以監聽並處理。

**這種方式適合：**
- 事件需要被多個 Service 消費的場景（例如：庫存更新、通知服務、分析服務都想知道同一筆訂單事件）
- Message Broker 對事件的**順序保證**做得比一般 Database 更好

**代價：**
- 在 Message Broker 上執行複雜查詢比 Database 更困難且不直覺

---

### Event Sourcing 帶來的附加好處：寫入效能提升

在傳統 CRUD 模式下，**寫密集工作負擔**（Write-intensive Workload）會導致大量 Contention——多個並發更新爭奪同一筆記錄的鎖，導致效能急劇下降。

**Event Sourcing 為什麼能緩解這個問題？**

每筆「變更」都變成一個 **Append-Only Event**——Append 操作**不需要任何資料庫鎖**，也比 UPDATE 操作更高效。

```
[傳統寫入]
UPDATE products SET inventory = inventory - 1 WHERE id = 'P001'
→ 需要對 P001 這筆記錄加鎖

[Event Sourcing 寫入]
INSERT INTO events (product_id, event_type, quantity) VALUES ('P001', 'PURCHASE', 1)
→ Append，不需要鎖
```

---

### 如何高效重建最新狀態：Snapshot + CQRS

每次都 replay 全部事件是不現實的——想像一個銀行帳戶從開戶到現在有數十年的交易記錄。對此有兩種優化策略：

#### 策略一：Snapshot（快照）

定期對實體的當前狀態建立 Snapshot：

```
[每月建立一次 Snapshot]
Snapshot-2026-03-31: Balance = $50,000

[查詢當前餘額時]
從最近的 Snapshot 開始，只 replay Snapshot 之後的事件
→ 不需要 replay 從帳戶開戶以來的所有事件
```

#### 策略二：CQRS + Read-Optimized Database

將 Event Sourcing 與 CQRS 結合——Command Side 專門負責 append events，而 Query Side 維護一份**預先計算好的當前狀態**：

```
[Command Side]
收到新事件 → Append 到 Event Log
         → 發佈到 Message Broker

[Query Side]
監聽 Message Broker 中的事件 Topic
→ 收到事件後更新 Read-optimized Database（甚至是 In-Memory Database）
→ 讀取速度極快
```

> **CQRS + Event Sourcing 是業界的黃金組合**：同時享有完整歷史稽核（Auditing）、高效寫入（Append-Only）、與極速讀取（Read-optimized DB）。

**代價依然是 Eventual Consistency**——從 Command 寫入到 Query Side 更新之間，存在時間窗口。

---

## 💡 重點摘要

- **Event Sourcing 不儲存「當前狀態」，而是儲存「所有事件的完整日誌」，狀態透過 replay 事件重建。**
- **事件是 Immutable 的——只能追加，無法修改或刪除。這保證了資料的完整不可偽造性。**
- **完整的事件日誌打開了 Auditing、欺詐偵測與消費行為分析的可能性，這是傳統 CRUD 模式無法提供的價值。**
- **寫密集工作負擔在 Event Sourcing 模式下受益於 Append-Only 的無鎖特性，大幅提升寫入效能。**
- **CQRS + Event Sourcing 是業界最佳實踐：用 Append-Only Event Log 做持久化，用 Read-Optimized Database 做高效查詢，但代價是犧牲強一致性，僅保證 Eventual Consistency。**

---

## 🔑 關鍵字

Event Sourcing, Immutable Events, Append-Only, Snapshot, CQRS, Event Replay, Eventual Consistency, Read-optimized Database, Audit Trail, Message Broker
