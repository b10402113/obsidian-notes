# Moderation Service 實作

## 📝 課程概述

本單元新增了 `moderation` service（Port 4003），它負責「言論審核」的商業邏輯——檢查留言內容是否包含特定禁詞（`orange`）。Moderation Service 完全**不處理任何前端請求**，只接收來自 Event Bus 的 `CommentCreated` 事件，根據內容決定是否標記為 `rejected`，然後發出 `CommentModerated` 事件通知其他 Service。

## 核心觀念與實作解析

### Moderation Service 的商業邏輯

```js
// moderation/index.js
app.post('/events', async (req, res) => {
  const { type, data } = req.body;

  if (type === 'CommentCreated') {
    const commentStatus =
      data.content.includes('orange') ? 'rejected' : 'approved';

    await axios.post('http://localhost:4005/events', {
      type: 'CommentModerated',
      data: {
        id: data.id,
        postId: data.postId,
        content: data.content,
        status: commentStatus
      }
    });
  }

  res.send({});
});
```

### 為什麼要引入 Moderation Service？

- **時效性需求**：審核邏輯可能隨時變更（禁詞清單更新）。如果把審核寫在 Comments Service，就每次改動都要重新部署 Comments Service——一個與「言論管理」核心功能無關的 Service。
- **異步處理**：真實場景中，AI 審核或人工審核可能耗時數分鐘到數小時，不能讓 `POST /posts/:id/comments` 等審核完成才回應。
- **關注點分離（Separation of Concerns）**：每個 Service 只管一件事。

### 留言的三種狀態

| 狀態 | 意義 | 前端呈現 |
|------|------|---------|
| `pending` | 審核中（Moderation Service 尚未處理） | 「此留言正在審核中...」 |
| `approved` | 審核通過 | 正常顯示留言內容 |
| `rejected` | 審核不通過（內容含禁詞） | 「此留言已被拒絕」 |

### 安裝與啟動

```bash
mkdir moderation && cd moderation
npm init -y
npm install express body-parser axios nodemon
# 注意：Moderation Service 不需要 cors（沒有前端直接請求它）
npm start  # 監聽 Port 4003
```

### 更新 Event Bus：加入 Moderation Service 的轉發

```js
// event-bus/index.js — POST /events handler 中追加：
axios.post('http://localhost:4003/events', event);  // moderation
```

Event Bus 是在 Moderation Service 建立之前就存在的，所以必須手動更新 Event Bus 的廣播清單。

## 💡 重點摘要

- **Moderation Service 完全不處理 HTTP 請求**——它只接收 Event Bus 的事件並發出事件，是純事件驅動的 Service。
- `orange` 只是教學用的禁詞測試——真實系統會從資料庫或設定檔讀取禁詞清單。
- **`CommentModerated` 事件**將「結果」（`approved` 或 `rejected`）廣播出去，讓所有感興趣的 Service 自行決定如何處理。
- Event Bus 必須在新增 Service 時**手動更新廣播清單**——這是手刻版本的限制，production 系統（如 NATS）有自動服務發現機制。

## 🔑 關鍵字

`Moderation Service`, `CommentModerated`, `Content Moderation`, `Business Logic Separation`, `Event-Driven`
