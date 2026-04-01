# 建立可重用的 NATS Listener 抽象類

## 📝 課程概述

本單元將 NATS 訂閱機制的繁瑣設定（42 行）封裝成一個可複用的 `Listener` 抽象類。我們採用 **Abstract Class** 而非直接實例化，是因為每個 Service 對同一事件的處理邏輯各不相同——Payment Service 收到 `ticket:created` 的處理方式，與 Order Service 必然不同。我們需要的是一個「藍圖」，讓每個 Service 去繼承並實作自己的 `onMessage` 商業邏輯。

---

## 核心觀念與實作解析

### 為什麼需要 Abstract Class？

在微服務中，幾乎每個 Service 都會需要監聽多個不同的事件。如果每次都寫 41 行的訂閱設定，會造成大量的 **Boilerplate Code**。更好的做法是：**把不變的基礎邏輯抽取出來，讓可變的商業邏輯由各 Service 自行實作**。

`Listener` 抽象類的設計如下：

| 抽象屬性 / 方法 | 用途 |
|---|---|
| `abstract subject: string` | 要監聽的 Channel 名稱（由子類定義） |
| `abstract queueGroupName: string` | Queue Group 名稱（由子類定義） |
| `abstract onMessage(data, msg): void` | 商業邏輯本體（由子類實作） |
| `ackWait: number` | 手動確認逾時時間，預設 5 秒（原本預設 30 秒） |
| `parseMessage(msg)` | 自動處理 `Buffer` 或 `String` 類型的訊息 |
| `listen()` | 建立 Subscription 並開始監聽 |

> **為什麼 `ackWait` 預設 5 秒而非 30 秒？** 30 秒太長了——如果商業邏輯執行失敗，我們希望盡快讓 NATS 重新投遞訊息，而不是等上半分鐘才重試。

### 程式碼解析

```typescript
// 抽象類本身不可直接使用
export abstract class Listener {
  abstract subject: string;
  abstract queueGroupName: string;
  abstract onMessage(data: any, msg: Message): void;

  protected ackWait = 5000; // 5 秒

  // helper: 建立 Subscription Options
  subscriptionOptions() {
    return this.client
      .subscriptionOptions()
      .setDeliverAllAvailable()
      .setManualAckMode(true)
      .setAckWait(this.ackWait)
      .setDurableName(this.queueGroupName);
  }

  listen() {
    const subscription = this.client.subscribe(
      this.subject,
      this.queueGroupName,
      this.subscriptionOptions()
    );

    subscription.on('message', (msg: Message) => {
      const data = this.parseMessage(msg);
      this.onMessage(data, msg);
    });
  }
}
```

### Queue Group 與 Durable Name 使用同一值

在 `setDurableName` 中直接使用 `queueGroupName` 的值，這是常見的設計選擇——讓同一組的消費者同時作為 Durable Name，**幾乎不需要讓兩者不同**。

### parseMessage 的設計

```typescript
private parseMessage(msg: Message): any {
  const data = msg.getData();
  return typeof data === 'string'
    ? JSON.parse(data)
    : JSON.parse(data.toString('utf-8'));
}
```

這個 helper 讓子類的 `onMessage` 永遠收到乾淨的 JavaScript Object，不需要自行處理 `Buffer` 與 `String` 的轉換。

---

## 💡 重點摘要

- **Abstract Class 是藍圖而非成品**——它定義了訂閱的完整流程，但真正的商業邏輯由子類實作。
- **`ackWait` 縮短為 5 秒可加速訊息重投**，比預設的 30 秒更適合需要快速重試的場景。
- **`setDurableName` 與 `queueGroupName` 使用同一值**，是最常見也最推薦的做法。
- **`parseMessage` helper 封裝了資料解析邏輯**，讓子類專注於商業邏輯。

---

## 🔑 關鍵字

Listener, Abstract Class, Queue Group, Durable Name, ackWait, Manual Ack Mode
