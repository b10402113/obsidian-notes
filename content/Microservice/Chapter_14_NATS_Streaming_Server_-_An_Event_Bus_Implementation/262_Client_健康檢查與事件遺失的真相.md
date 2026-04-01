# Client 健康檢查與事件遺失的真相

## 📝 課程概述

本單元透過實際觀察 NATS Streaming 的監控頁面，解釋為什麼在重啟 Listener 時，剛發布的事件會暫時「消失」，遲遲不被接收。問題的根源在於 Server 的**心跳檢查（Heartbeat）**機制——Server 在 Client 斷線後不會立即知道，而是要等到多次心跳失敗後才會將 Subscription 從清單中移除。

---

## 核心觀念與實作解析

### 觀察問題：重啟後事件延遲出現

如果你快速重啟 Listener 並立即發布事件，可能會發現事件**沒有立即出現**，要等大約 30 秒才會出現。原因是：

1. Listener 被關閉後，**Server 仍然認為該 Client 還活著**
2. Server 嘗試發送事件給這個「已死亡」的 Client，但發送失敗
3. Server 等待 ack 超時（30 秒）後，才將事件重新發送給另一個實例

---

### 為什麼 Server 不立即知道 Client 離線？

Client 進程被殺掉後，TCP 連線不會立即中斷——網路連線本身需要時間才能偵測到問題。這段時間內，Server 完全不知道 Client 已經消失了。

---

### 心跳檢查機制（Heartbeat）

NATS Streaming Server 會定期對所有已連線的 Client 發送**心跳請求（Heartbeat Request）**，確認它們是否還在運行。相關參數設定：

| 參數 | 意義 |
|------|------|
| `-hbi` | Heartbeat Interval — Server 多久發送一次心跳（預設值） |
| `-hbt` | Heartbeat Timeout — Client 必須在多久內回覆心跳 |
| `-hbpf` | Heartbeat Fail — Client 必須連續失敗幾次心跳才會被認定為離線 |

> **實質效果**：即使 Client 當機了，Server 也要等到連續失敗數次心跳後才會「放棄」並從訂閱清單中移除。這導致了事件在 Client 離線後仍然會「卡在 Server 端」長達數秒甚至更久。

---

### 透過監控頁面觀察 Client 狀態

NATS Streaming 提供了 HTTP 監控頁面（`http://localhost:8222`），可以查看伺服器內部的所有狀態：

```
http://localhost:8222/streaming/channels/subs?subs=1
```

這個端點會顯示：
- 每個 Channel 上的 Subscription 清單
- 每個 Subscription 的 Client ID
- `is_offline` 欄位 — 標示該 Client 是否已離線但尚未被清除
- `ack_awaiting_time` — Server 在等待 ack 的時間

當 Client 離線但尚未被清除時，`is_offline` 為 `false`（Server 仍認為它在線上）。大約 30 秒後，Subscription 才會從清單中消失。

---

### 心跳參數與事件遺失時間的關係

事件在 Listener 離線後「消失」的時間，大致等於：

```
max(心跳清除時間, 30秒等待 ack)
```

即使我們將心跳頻率調高、減少失敗容忍次數，仍然無法完全消除這個延遲。**因此我們需要另一種方式：優雅的 Client 關閉（Graceful Shutdown）**，這是下一個單元的重點。

---

## 💡 重點摘要

- **Server 不會立即知道 Client 已離線，TCP 連線需要時間才能偵測到中斷。**
- **Heartbeat 機制讓 Server 能自動偵測離線 Client，但需要等待多次失敗後才會真正移除。**
- **30 秒的 ack 等待時間 + 心跳延遲，導致離線期間的事件會延遲出現，而非立即消失。**
- **心跳設定（`-hbi`, `-hbt`, `-hbpf`）可以調整，但無法完全消除事件延遲；需要從 Client 端主動告知 Server 關閉。**

---

## 🔑 關鍵字

Heartbeat, Monitoring, is_offline, ack_awaiting_time, TCP Connection, Client Health Check
