# 等待事件發布完成：async/await 化 publish

## 📝 課程概述

本單元將 `publish` 從**不回傳值**的同步函式，升級為**回傳 Promise 並支援 async/await** 的非同步函式。背後的原因是：事件發布是一個網路 IO 操作，有失敗的可能。呼叫方需要知道發布是否成功，才能決定是否繼續執行後續邏輯（例如：更新資料庫後才發布 Event，失敗時應觸發 rollback）。

---

## 核心觀念與實作解析

### 為什麼需要 async publish？

```typescript
// 之前：fire-and-forget，無法知道是否成功
publisher.publish(data);

// 之後：可等待結果，失敗時可處理
try {
  await publisher.publish(data);
  // 只有發布成功後才執行這裡
} catch (err) {
  console.error('發布失敗:', err);
  // 執行 rollback 或其他補償邏輯
}
```

### Promise 包裝 callback

NATS client 的 `publish` 使用 callback 語法：

```typescript
client.publish(subject, data, (err) => {
  if (err) {
    // 發布失敗
  } else {
    // 發布成功
  }
});
```

我們將其包裝為 Promise：

```typescript
publish(data: T['data']): Promise<void> {
  return new Promise((resolve, reject) => {
    this.client.publish(
      this.subject,
      JSON.stringify(data),
      (err) => {
        if (err) {
          reject(err);  // 發布失敗 → reject
        } else {
          resolve();    // 發布成功 → resolve
        }
      }
    );
  });
}
```

> **這是一個經典的 Callback-to-Promise 轉換模式**。將底層 callback API 包裝後，呼叫方就能使用乾淨的 async/await 語法，而不需要理會底層實作細節。

### 錯誤處理流程

```
事件發布失敗
    │
    ├── reject(Promise)
    │       │
    │       └── await 處拋出 Error → catch(err) → rollback
    │
    └── resolve()
            │
            └── await 之後繼續執行 → 確認業務流程完成
```

---

## 💡 重點摘要

- **Async publish 的核心價值在於錯誤處理**：失敗時呼叫方可以執行 rollback 或重試邏輯，而非假設發布成功。
- **Callback-to-Promise 包裝是將舊式 API 現代化的常見技巧**，讓新舊程式碼和諧共存。
- **`reject(err)` 的設計讓錯誤會自動向上傳播到 `await` 表達式**，不需要手動處理。

---

## 🔑 關鍵字

async/await, Promise, Callback-to-Promise, NATS, publish, Error Handling
