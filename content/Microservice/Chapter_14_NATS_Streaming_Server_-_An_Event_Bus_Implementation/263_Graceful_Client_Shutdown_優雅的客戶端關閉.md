# Graceful Client Shutdown：優雅的客戶端關閉

## 📝 課程概述

本單元說明如何**主動通知 NATS Server「我要關閉了」**，以避免 Server 繼續嘗試向已斷線的 Client 發送事件。我們會實作作業系統訊號（Signal）攔截機制——當程式收到 `SIGINT`（Ctrl+C）或 `SIGTERM` 訊號時，先呼叫 `stan.close()` 通知 Server，再正常退出程式。

---

## 核心觀念與實作解析

### 為什麼需要主動關閉？

即使 Client 當機，Server 也要等到 Heartbeat 多次失敗後才會將 Subscription 從清單中移除。在這段時間內，發布的事件會「卡住」無法被正確處理。

**解決方案**：在程式真正退出之前，主動通知 Server：「我即將離線，請不要再發送任何事件給我。」這就是 `stan.close()` 的作用。

---

### 實作：攔截作業系統訊號

```typescript
// 監聽「即將關閉」事件
stan.on('close', () => {
  console.log('NATS connection closed');
  process.exit();
});

// 攔截 Ctrl+C (SIGINT) 與 終止訊號 (SIGTERM)
process.on('SIGINT', () => stan.close());
process.on('SIGTERM', () => stan.close());
```

**流程解析：**

```
使用者按 Ctrl+C
  │
  ▼
觸發 SIGINT 訊號
  │
  ▼
呼叫 stan.close()
  │
  ▼
Client 向 Server 發送關閉請求：「我即將離線」
  │
  ▼
Server 立即從訂閱清單中移除該 Client
  │
  ▼
Client 執行 console.log + process.exit()
```

---

### 驗證效果

加入 `stan.close()` 處理後，立即刷新監控頁面，會看到 Subscription 清單**立即從 3 個變回 2 個**（而不是等待 30 秒）。

但仍有例外——如果使用**強制關閉（Force Kill）**，作業系統訊號處理器根本來不及執行，這個機制就失效了。這也是為什麼 Heartbeat 機制仍然不可或缺。

---

### Windows 的限制

> `SIGINT` 與 `SIGTERM` 是 Unix/Linux/macOS 的標準訊號。**Windows 並不總是使用相同的訊號**，因此在 Windows 環境下，這個 graceful shutdown 機制可能無法正常運作。這點在生產環境中需要注意。

---

### 訊號說明

| 訊號 | 觸發時機 |
|------|----------|
| `SIGINT` | 使用者按下 `Ctrl+C` |
| `SIGTERM` | `ts-node-dev` 自動重啟程式時（或其他程式送出終止請求） |
| Force Kill | 直接從行程管理器或 `kill -9` 強制終止，**訊號處理器來不及執行** |

---

## 💡 重點摘要

- **主動呼叫 `stan.close()` 可以立即通知 Server 移除 Subscription，避免 Server 在 Heartbeat 偵測到失敗前就嘗試發送事件。**
- **`SIGINT` 與 `SIGTERM` 訊號處理器確保在程式結束前先完成 graceful shutdown。**
- **Force Kill (`kill -9`) 會繞過訊號處理器，心跳機制仍是最後的安全網。**
- **Windows 不完全支援標準 Unix 訊號，生產環境需注意平台差異。**

---

## 🔑 關鍵字

Graceful Shutdown, stan.close, SIGINT, SIGTERM, Force Kill, Signal Handler
