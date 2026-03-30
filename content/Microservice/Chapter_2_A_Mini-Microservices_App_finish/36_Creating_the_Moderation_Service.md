# Moderation Service 實作：接收 CommentCreated、發出 CommentModerated

## 📝 課程概述

本單元實作第四個 Service——**Moderation Service**（Listen on Port 4003）。它的職責是：監聽 `CommentCreated` 事件，檢查留言內容是否包含敏感字（"orange"），根據結果發出 `CommentModerated` 事件，並由 Comment Service 負責處理這個事件，更新留言狀態後再發出通用的 `CommentUpdated` 事件。

---

## 核心觀念與實作解析

### Moderation Service 的職責

- **監聽**：`CommentCreated` 事件（來自 Event Bus）
- **處理**：檢查留言內容，決定 `approved` 或 `rejected`
- **發射**：`CommentModerated` 事件

### Moderation Service 的 API 設計

Moderation Service **不需要 CORS**，因為它不接受來自 React App 的直接 HTTP 請求。它只需要一個端點：

| Method | Path | 功能 |
|--------|------|------|
| `POST` | `/events` | 接收 Event Bus 廣播的事件 |

### 實作重點：事件處理

```javascript
app.post('/events', (req, res) => {
  const { type, data } = req.body;

  if (type === 'CommentCreated') {
    const { id, content, postId } = data;
    const status = content.includes('orange') ? 'rejected' : 'approved';

    // 發射 CommentModerated 事件
    await axios.post('http://localhost:4005/events', {
      type: 'CommentModerated',
      data: { id, content, postId, status }
    });
  }

  res.send({});
});
```

> 這裡的 `await` 確保事件發射完成後才回傳 response。

### 為什麼 CommentModerated 要由 Comment Service 處理？

根據 Option 3 的設計，`CommentModerated` 是一個**專業的 Comment 更新事件**，只有 Comment Service（Comment 的 Domain Owner）知道如何處理它。Moderation Service 只是 Moderation 業務邏輯的執行者，但 Comment Service 才是最終決定「這筆 Comment 的狀態變成什麼」的權威。

流程是這樣的：
1. Moderation Service → emit `CommentModerated`
2. Event Bus → 廣播給 Comment Service
3. Comment Service → 收到後更新本地 comment 狀態 → emit `CommentUpdated`
4. Event Bus → 廣播給 Query Service
5. Query Service → 收到 `CommentUpdated`，用 attributes 直接覆蓋本地記錄

---

### 為 CommentService 加上 Status 欄位

Comment Service 在建立留言時，需要多儲存一個 `status` 欄位：

```javascript
// Comment Service: POST /posts/:id/comments
const newComment = {
  id: commentId,
  content,
  status: 'pending'  // 新增：預設為待審核
};
```

`CommentCreated` 事件也需要攜帶這個 `status` 欄位，讓 Query Service 能在第一時間知道留言處於「待審核」狀態。

---

## 💡 重點摘要

- Moderation Service 是一個**純事件驅動**的 Service——它只接收事件，處理業務邏輯，再發射新的事件，不需要 CORS。
- Moderation Service 本身不做資料持久化——它只決定「留言是通過還是拒絕」，實際的資料更新交給 Comment Service。
- `status: 'pending'` 在 Comment Service 建立的同時就寫入，確保 Query Service 能即時呈現「待審核」狀態。

## 關鍵字

Moderation Service, CommentCreated, CommentModerated, Status Field, pending, approved, rejected, Domain-Owned Event, Event Processing, Port 4003, Event Bus Emission
