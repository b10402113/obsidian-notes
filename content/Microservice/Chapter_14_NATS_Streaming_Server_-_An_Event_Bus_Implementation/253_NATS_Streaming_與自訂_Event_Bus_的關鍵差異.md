# NATS Streaming 與自訂 Event Bus 的關鍵差異

## 📝 課程概述

本單元從高層次宏觀比較**自訂 Event Bus**（第一個練習專案中用 Express + Axios 打造的簡易版本）與**NATS Streaming Server** 之間的根本差異。我們會聚焦在三件事：Client 溝通方式、Channel/Topic 訂閱模型，以及事件持久化機制。掌握這些差異是理解後續所有實作細節的基礎。

---

## 核心觀念與實作解析

### 差異一：通訊方式的演進

**自訂 Event Bus：**

```
Service → Axios POST → Custom Event Bus (Express) → Axios POST → Target Service
```

這整個鏈路的本質是：**普通的 HTTP 請求轉發**。我們只是把 Express 拿來轉發 JSON，沒有任何額外的訊息管理機制。

**NATS Streaming：**

```
Service → node-nats-streaming → NATS Streaming Server → node-nats-streaming → Target Service
```

我們不再使用 Axios 或 Express 處理事件。取而代之的是專門為 NATS Streaming 設計的 Client Library（`node-nats-streaming`），它提供了事件導向的 API，**大量依賴 Callback** 而非 `async/await`，這點在剛開始接觸時需要適應。

> 文件說明：NATS Streaming 的文件和 node-nats-streaming Library 的 API 都需要仔細閱讀——**從一開始就需要理解內部機制**，否則後續很容易出錯。

---

### 差異二：事件廣播範圍——從「全部發送」到「精準訂閱」

**自訂 Event Bus 的廣播行為：**

發出一個事件時，**所有 Service 都會收到**。即使發出事件的 Service 本身也會收到一份（技術上如此，雖然通常沒有實際用途）。

**NATS Streaming 的 Topic/Channel 訂閱模型：**

```
Service A (Ticket Service)
    │
    │  Publish: ticket:created
    ▼
NATS Streaming Server (Channel: ticket:created)
    │
    ├──▶ Order Service (訂閱 ticket:created) ✅ 收到
    └──▶ Payment Service (訂閱 payment:created) ❌ 沒興趣，不會收到
```

在 NATS Streaming 中，事件不會自動發給所有人。我們必須：

1. **定義 Channel（也稱為 Topic）** — 代表一種類型的事件（如 `ticket:created`）
2. **各 Service 主動訂閱** 它們關心的 Channel
3. 事件**只會被送到有訂閱的 Service**

這大幅降低了系統的耦合度：Ticket Service 不需要知道有多少 Service 會處理它的事件，只需要專心發佈到對應的 Channel。

---

### 差異三：事件持久化——記憶體陣列到可設定的後端儲存

**自訂 Event Bus 的持久化：**

我們在手動打造的 Event Bus 中，用一個**記憶體陣列**儲存所有曾經發出過的事件。這是為了讓離線的 Service 重新上線後，能夠取得所有遺漏的事件。

**NATS Streaming 的持久化機制：**

NATS Streaming Server 預設將所有事件儲存在**記憶體**中，但可以進一步設定：

- **記憶體（預設）** — 速度最快，但 Server 重啟後事件全失
- **檔案儲存（Flat Files）** — 事件寫入硬碟，Server 重啟後可復原
- **資料庫（MySQL / PostgreSQL）** — 更適合生產環境的長期持久化

> **為什麼這很重要？** 當 NATS Streaming Server 因故需要重啟時，如果有設定檔案或資料庫持久化，它就能在重啟後立即繼續提供完整的歷史事件給新上線或重新連線的 Service。

---

### 重要術語對照

| 自訂 Event Bus 說法 | NATS Streaming 說法 |
|---------------------|---------------------|
| 發送事件 | Publish |
| 接收事件 | Subscribe |
| 事件列表（記憶體陣列） | Channel / Topic |
| 事件內容 | Message |

> 在 NATS 生態中，「事件」通常被稱為 **Message**，而「頻道」稱為 **Subject** 或 **Channel**。課後複習文件時請注意這點。

---

## 💡 重點摘要

- **NATS Streaming 使用專門的 Client Library（node-nats-streaming），不再以 Axios/Express 處理事件。**
- **事件預設不再發給所有 Service，各 Service 必須主動訂閱感興趣的 Channel（Topic）。**
- **NATS Streaming 內建事件持久化，可設定為記憶體、檔案或資料庫儲存。**
- **自訂 Event Bus 的「全部廣播」模型在規模擴大時會造成大量不必要的處理負擔。**

---

## 🔑 關鍵字

node-nats-streaming, Channel, Topic, Publish, Subscribe, Persistence, Subject
