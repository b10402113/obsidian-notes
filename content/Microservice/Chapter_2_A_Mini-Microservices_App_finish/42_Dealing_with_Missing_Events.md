# Event 遺失問題：服務離線與新服務上線的挑戰

## 📝 課程概述

本單元分析事件驅動架構中最核心的可靠性問題——**當某個 Service 離線一段時間後，它會錯過這段期間發出的所有事件**，導致本地資料與系統真實狀態產生永久不一致。我們會分析三種可能的解決方案，最終選擇第三種：**讓 Event Bus 儲存所有歷史事件，讓各 Service 在上線時可以主動同步漏掉的事件**。

---

## 核心觀念與實作解析

### 問題一：Service 臨時離線

```
時間軸：───[Moderation 在線]───[離線]───[重新上線]───
事件：      Event1         Event2 Event3     Event4
                         ↑   ↑
                    這兩個事件 Moderation 完全錯過
```

當 Moderation Service 恢復上線後，Event 2 和 Event 3 已經在 Event Bus 中消失了。Moderation Service 不知道有這兩個事件發生，永遠無法處理它們。

### 問題二：新 Service 延後上線

若 Query Service 在系統運行一年後才被加入，它從一開始就錯過了所有歷史事件，根本無法有任何資料。

---

### Option 1：使用同步 Request 進行資料同步

新 Service 上線時，直接向其他 Service 發送同步 HTTP 請求取得所有歷史資料：
- **缺點**：需要各 Service 額外實作「批量匯出所有資料」的 API 端點
- **缺點**：等於又回到了同步通訊的依賴問題

### Option 2：讓新 Service 直接存取其他 Service 的 Database

讓 Query Service 直接連接到 Post Service 和 Comment Service 的資料庫：
- **缺點**：違反了「Service 不能直接讀取其他 Service 的 Database」的紀律
- **缺點**：若使用了不同類型的資料庫（MySQL + MongoDB），新 Service 需要實作多種資料庫驅動程式

### Option 3：Event Bus 儲存所有歷史事件（最終選擇）

Event Bus 不只是轉發事件，還會將**每個收到的事件**存入自己的資料庫（陣列）。當任何 Service 上線時，都可以向 Event Bus 請求「所有歷史事件」，並用自己的事件處理邏輯一一 replay：

```
Event Bus 儲存：
Event1 → { type: 'PostCreated', data: {...} }
Event2 → { type: 'CommentCreated', data: {...} }
Event3 → { type: 'CommentModerated', data: {...} }
Event4 → { type: 'CommentUpdated', data: {...} }

Query Service 上線時：
→ GET /events → 收到 [Event1, Event2, Event3, Event4]
→ 對每個事件執行 handleEvent(type, data)
→ 本地資料完全與系統同步
```

當 Moderation Service 離線後重新上線，它只需要向 Event Bus 請求「最後一個已處理事件之後的所有事件」即可追平進度。

### ⚠️ 這個方案的代價

Event Bus 的資料儲存會隨著時間不斷增長，可能達到非常大的規模。但就像第一章計算的那樣，雲端儲存的成本已經非常低——這個代價是完全值得的。

---

## 💡 重點摘要

- 事件驅動架構最大的挑戰是：**事件可能遺失**——當 Service 離線或新加入時，歷史事件無法被處理。
- Option 1 和 Option 2 都涉及程度不等的同步依賴或違反架構紀律的設計。
- **Option 3 的核心思想是：Event Bus 成為所有事件的「永久記錄者」**，每個 Service 都可以透過 replay 歷史事件來追上系統狀態。
- 這個問題在後續的大型專案中，會透過 Kafka 或 NATS 這類專業的 Message Broker 來自動處理。

## 關鍵字

Missing Events, Event Replay, Service Offline, Event Store, Event Bus Storage, Option 1 Sync, Option 2 Direct DB Access, Option 3 Event Store, Event Synchronization, Historical Events, Message Broker
