# 更新 Common Module：整合事件系統

## 📝 課程概述

本單元將前面章節建立的 `BaseListener`、`BasePublisher`、`Subjects Enum`、`TicketCreatedEvent` 與 `TicketUpdatedEvent` 全部遷移到 `common` 模組中。這是微服務架構的關鍵設計原則：**所有 Service 共享同一份 Event 結構定義**，確保每個 Service 對 Subject 名稱與資料結構的理解完全一致，從根本上杜絕跨 Service 的型別不一致問題。

---

## 核心觀念與實作解析

### Common Module 的最終結構

```
common/src/events/
  BaseListener.ts         ← 抽象 Listener 類
  BasePublisher.ts        ← 抽象 Publisher 類
  subjects.ts             ← Subjects Enum（含 TicketUpdated）
  TicketCreatedEvent.ts   ← ticket:created 事件介面
  TicketUpdatedEvent.ts   ← ticket:updated 事件介面
common/src/index.ts       ← 統一 export 所有 events
```

### Subject 的更新：加入 TicketUpdated

```typescript
export enum Subjects {
  TicketCreated = 'ticket:created',
  TicketUpdated = 'ticket:updated'
}
```

`order:updated` 被移除，因為尚未實際使用。**保持 Enum 與實際已實作的事件同步**，避免遺留虛構的 Subject 造成混淆。

### Event 介面中加入 userId

```typescript
export interface TicketCreatedEvent {
  subject: Subjects.TicketCreated;
  data: {
    id: string;
    title: string;
    price: number;
    userId: string;  // ✅ 新增：Ticket 的擁有者 ID
  };
}
```

> **為什麼要加 `userId`？** 因為其他 Service（如 Payment Service）收到 `ticket:created` 時，很可能就是需要知道「誰建立了這個 Ticket」才能執行後續業務邏輯。Event 的資料結構應涵蓋所有消費者的需求。

### 程式碼歸屬原則

| 檔案 | 放在 Common Module？ | 放在 Service 內部？ |
|---|---|---|
| `BaseListener` / `BasePublisher` | ✅ | |
| `Subjects` Enum | ✅ | |
| Event Interfaces (`TicketCreatedEvent` 等) | ✅ | |
| Service 特定的 Listener（如 `TicketCreatedListener`） | | ✅ |
| Service 特定的 Publisher（如 `TicketCreatedPublisher`） | | ✅ |

### 為什麼每個 Service 要自己實作 Listener/ Publisher 子類？

同一個 `ticket:created` Event：
- **Payment Service**：收到後需要驗證付款、建立訂單
- **Email Service**：收到後需要發送通知 Email
- **Inventory Service**：收到後需要更新庫存

商業邏輯完全不同，所以**子類必須由各 Service 自行實作**，不能放在 Common Module。

### NATS Pod 的資料清理

測試階段 NATS 中累積了大量測試資料（無效的 ticket ID 等）。重啟 Pod 會清除所有歷史訊息：

```bash
kubectl get pods  # 找到 NATS pod 名稱
kubectl delete pod <pod-name>  # Deployment 會自動重建新 Pod
```

> **NATS 預設將訊息存於記憶體**，重啟後所有歷史 Event 消失。這對開發/測試階段非常友好，但 Production 環境需注意資料持久化設定。

### 跨語言微服務的替代方案

如果你的微服務使用多種語言（Java、Python、Go 等），**TypeScript Enum 無法跨語言共用**。此時可以考慮：

- **JSON Schema**：以 JSON 格式定義 Event 結構，各語言皆有對應驗證庫
- **Protocol Buffers (ProtoBuf)**：二進位序列化格式，支援跨語言定義（`.proto` 檔）
- **Apache Avro**：以 JSON 或 JSON Schema 定義結構，強於 Java 生態系

> 授課講師提到他曾嘗試用 JSON Schema 與 ProtoBuf 建構同一套系統，**最終認為 TypeScript + Common Module 的方案最為直觀且易用**。

---

## 💡 重點摘要

- **Common Module 是微服務 Event 系統的「單一事實來源」**——所有 Service 從同一處取得 Event 結構定義。
- **Event 介面的資料欄位要涵蓋所有消費者的需求**，`userId` 這類欄位應提前加入，避免日後 API 破壞性變更。
- **Listener/Publisher 的子類放在 Service 內部**，因為同一 Event 在不同 Service 的處理邏輯必然不同。
- **NATS 訊息存於記憶體**，重啟 Pod 即清除所有歷史 Event——開發階段需注意。

---

## 🔑 關鍵字

common module, Subjects, TicketUpdatedEvent, cross-service, JSON Schema, Protocol Buffers, NATS
