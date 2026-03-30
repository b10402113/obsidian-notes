# Option 2 與 Option 3：Domain-Owned Event 模式

## 📝 課程概述

本單元繼續分析 Moderation 功能的實作方案。我們發現 Option 1 會導致使用者提交留言後立即看不到該留言。Option 2 解決了這個 Immediate Feedback 問題，但引入了另一個長期維護性問題——**Query Service 必須知道如何處理所有可能的 Comment 更新方式**。最終，我們採用 Option 3 的核心思想：**由「擁有該 Resource 的 Service」負責處理所有專業的更新操作，再以通用事件通知其他 Service**。

---

## 核心觀念與實作解析

### Option 2：Query Service 即時收到 CommentCreated 事件

```
User Submit → Comment Service → emit CommentCreated
                    ↓                    ↓
             (寫入本地資料)        Moderation Service
                                           ↓
                            emit CommentModerated (攜帶 status)
                                           ↓
                              Query Service 收到並更新狀態
                    ↓
Query Service 立即收到 CommentCreated（status: pending）→ 使用者立即看到「待審核」狀態
```

**Option 2 的優點**：使用者提交留言後，Query Service 立即收到 `CommentCreated` 事件，馬上可以呈現「待審核」狀態。使用者整理頁面不會感到困惑。

**Option 2 的缺點**：Query Service 必須**知道如何處理 `CommentModerated` 事件**。隨著業務發展，Comment 可能會有越來越多的更新方式（moderation、upvote/downvote、promote、mark as anonymous、mark as advertised...）。每一種更新方式都代表一種新的事件類型，如果全部由 Query Service 處理，Query Service 就必須**知道所有 Comment 的業務邏輯**——這完全破壞了 Service 的邊界封裝原則。

---

### Option 3：Domain-Owned Event Pattern（最終選擇）

核心思想：**讓「擁有該 Resource 的 Service」負責處理所有針對該 Resource 的專業操作，其他 Service 只接收通用的「已更新」通知**。

#### Comment Service 是 Comment 的 Domain Owner

Comment Service 是唯一有權決定「Comment 的結構是什麼」以及「Comment 可以怎麼被更新」的 Service。其他 Service（如 Query Service）不應該知道 Comment 的內部更新邏輯。

#### Option 3 的事件流程

```
User Submit → Comment Service → emit CommentCreated (status: pending)
                                    ↓
                    ┌───────────────┴───────────────┐
                    ↓                               ↓
              Query Service                   Moderation Service
           (立即看到新留言)                    (執行審核邏輯)
                                                  ↓
                                    emit CommentModerated (攜帶完整 updated comment)
                                                  ↓
                                         Comment Service
                                    (唯一知道如何處理此事件)
                                    (更新本地 comment 狀態)
                                                  ↓
                                    emit CommentUpdated (通用更新事件)
                                                  ↓
                                         Query Service
                                    (只知道 comment 被更新了，
                                     直接用 event data 覆蓋本地記錄)
```

#### 為什麼需要 CommentUpdated 通用事件？

Query Service **不需要知道** Comment 是被 Moderation 更新、被 Promote、還是被 Downvote。它只需要知道：「這筆 Comment 有更新了，把 attributes 拿去更新本地記錄吧」。`CommentUpdated` 事件就像一張「更新通知卡」，讓 Query Service 不需要理解背後的 business logic：

```javascript
// Query Service 收到 CommentUpdated
app.post('/events', (req, res) => {
  const { type, data } = req.body;

  if (type === 'CommentUpdated') {
    // 直接用 event 提供的所有 attributes 覆蓋本地記錄
    const { id, content, status, postId } = data;
    const post = posts[postId];
    const commentIndex = post.comments.findIndex(c => c.id === id);
    post.comments[commentIndex] = { id, content, status };
  }
  // ...
});
```

---

### Domain-Owned Event Pattern 的核心原則

1. **每個 Resource 只有一個 Domain Owner**：Comment 的 Domain Owner 是 Comment Service，只有它知道 Comment 的完整結構與所有更新方式
2. **Domain Owner 處理所有專門的事件**：Moderation 結果由 Comment Service 處理，而不是由 Query Service 直接處理
3. **Domain Owner 發出通用更新事件**：更新完成後，發出 `CommentUpdated`，其他 Service 只需要「接受這個更新」，不需要理解為什麼要更新

---

## 💡 重點摘要

- Option 2 解決了使用者體驗問題，但讓 Query Service 承擔了不屬於它的業務邏輯——長期維護成本極高。
- Option 3 的核心精神是：**讓每個 Service 只做自己擅長的事**。Query Service 只管呈現資料；Comment Service 才是 Comment 資料的專家。
- `CommentUpdated` 通用事件的設計，讓 Query Service 不需要隨著 Comment 更新方式的增加而不斷修改——它永遠只需要接收 attributes 並覆蓋。
- 這個模式在後續的大型專案中會反覆出現，是 microservices 設計中非常重要的 pattern。

## 關鍵字

Domain-Owned Event Pattern, Comment Service Domain Owner, CommentUpdated, Generic Event, Domain Logic, Separation of Concerns, Option 3, Business Logic Encapsulation, CommentModerated, Event-Driven Architecture, Microservices Design Pattern
