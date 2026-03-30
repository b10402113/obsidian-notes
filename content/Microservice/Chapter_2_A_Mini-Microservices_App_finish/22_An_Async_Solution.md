#  Async Solution：Query Service + Event Bus 模式

## 📝 課程概述

本單元介紹第二種解決方案——**Async（事件驅動）模式**。我們會建立一個全新的 `Query Service`，專門用來儲存「已組合好的」Posts + Comments 資料，並透過 Event Bus 在背景接收來自 Post Service 與 Comment Service 的事件，自動維護這份組合資料。這使得 React App 只需要對 Query Service 發送**一次**請求，就能取得所有需要的資料。

---

## 核心觀念與實作解析

### 解決方案的核心理念

在 Post Service 和 Comment Service 之間，引入第三個 Service——**Query Service**。

Query Service 的職責：
1. 接收所有 Service 發出的事件
2. 維護一份已組合好的資料結構（Posts + 各自關聯的 Comments）
3. 提供一個端點，讓 React App **一次請求**就能取得所有資料

### Query Service 的資料結構

```javascript
{
  "post-id-1": {
    id: "post-id-1",
    title: "New Post",
    comments: [
      { id: "comment-id-1", content: "Great post!" }
    ]
  }
}
```

每一篇 Post 都有自己的 `comments` 陣列。Query Service 就是這份已「預先組合」好的資料的持有者。

---

### Event Bus 的角色

Event Bus 在這裡扮演**事件轉發中心**：

```
Post Service ──emit──→ Event Bus ──echo──→ Query Service
                                          │
Comment Service ──emit──→ Event Bus ──echo──→ Query Service
```

當任何一個 Service 向 Event Bus 發送事件時，Event Bus 會將這個事件**原封不動地廣播給所有其他 Service**，包括 Query Service。

---

### 事件驅動的工作流程

#### 流程一：建立文章

1. User 對 Post Service 發送 `POST /posts`
2. Post Service 存入本地資料庫，**同時**發出事件：`{ type: 'PostCreated', data: { id, title } }` 給 Event Bus
3. Event Bus 將事件 echo 給 Query Service
4. Query Service 收到事件後，在自己的資料結構中建立新文章記錄，`comments` 初始化為空陣列

#### 流程二：建立留言

1. User 對 Comment Service 發送 `POST /posts/:id/comments`
2. Comment Service 存入本地資料庫，**同時**發出事件：`{ type: 'CommentCreated', data: { id, content, postId } }` 給 Event Bus
3. Event Bus 將事件 echo 給 Query Service
4. Query Service 收到事件後，找到對應的 Post ID，將新留言 `push` 進該文章的 `comments` 陣列

#### 流程三：Client 取得完整資料

React App 對 Query Service 發送一次 GET 請求，直接取得 `{ postsMap }`——包含所有文章與其所有留言，**一次請求，解決所有問題**。

---

### Pros & Cons 再次回顧

**優點**：
- Query Service **零依賴**：即使 Post Service 或 Comment Service 全掛，Query Service 仍能正常回傳已緩存的資料。
- **極速回應**：所有資料都在本地，沒有跨 Service 的網路呼叫延遲。
- 消滅了 N+1 問題：client 只需要一次請求。

**缺點**：
- **資料重複**：同一份資料存在多個 Service 中。但儲存成本極低，這不是真正的問題。
- **概念複雜度較高**：需要理解事件的發送、監聽、處理的整個生命週期。
- 事件驅動架構有其獨特的 corner cases（例如：事件遺失、重複處理、併發問題），這些會在後續章節深入討論。

---

## 💡 重點摘要

- Query Service 是整個模式的關鍵——它負責維護一份「已組合好」的資料副本，讓 client 可以用一次請求完成所有查詢。
- Event Bus 在這裡的角色非常簡單：**接收事件，原封不動地 echo 給所有已連接的 Service**。
- 這個模式的代價是資料會在多個 Service 中重複出現，但換來的是**零依賴關係與極高的查詢效能**。
- 這個迷你專案中刻意將 Post 和 Comment 分為兩個 Service，是為了**學習目的**。實際上，如果只有這兩個 Resource，通常會放在同一個 Service 中。

## 關鍵字

Query Service, Event Bus, Event-Driven Architecture, PostCreated, CommentCreated, Data Duplication, Zero Dependency, Event Echo, In-Memory Storage, Asynchronous Communication, N+1 Problem Solution
