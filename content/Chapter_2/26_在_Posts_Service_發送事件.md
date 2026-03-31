# 在 Posts Service 發送事件

## 📝 課程概述

本單元修改了 `posts` service 與 `comments` service，讓它們在執行「建立文章」或「建立留言」時，主動透過 Axios 向 Event Bus（Port 4005）發送事件。這是微服務架構中「事件驅動」通訊模式的起點。

## 核心觀念與實作解析

### Posts Service 發送 `PostCreated` 事件

在 `posts/index.js` 的 `POST /posts` 處理函式中，建立完文章後立即發送事件：

```js
app.post('/posts', async (req, res) => {
  const id = crypto.randomBytes(4).toString('hex');
  const { title } = req.body;
  posts[id] = { id, title };

  // 發送事件到 Event Bus
  await axios.post('http://localhost:4005/events', {
    type: 'PostCreated',
    data: { id, title }
  });

  res.status(201).send(posts[id]);
});
```

### Comments Service 發送 `CommentCreated` 事件

```js
// comments/index.js
await axios.post('http://localhost:4005/events', {
  type: 'CommentCreated',
  data: { id: commentId, content, postId }
});
```

### 事件發送的時機

事件應該在**資料變更成功之後**才發送。如果先發送事件，Event Bus 廣播出去後，某些 Service 嘗試處理時可能會發現資料尚未存在而失敗。因此「先寫入，再廣播」是標準順序。

### 為什麼有些 Service 不感興趣的事件也會收到？

Event Bus 目前是**無條件廣播**——它把所有收到的 `POST /events` payload 原封不動轉發給所有 Service。這不是最有效率的設計（production 系統通常有訂閱機制），但作為教學用途，這樣的簡化模型更容易理解訊息流向。

### 初期測試時的 404 錯誤

在所有 Service 尚未實作 `POST /events` 端點前，Event Bus 廣播時對目標 Service 的 `POST /events` 請求會得到 **404 Not Found**。這是預期行為——下個單元就會在各 Service 加入事件接收處理器。

## 💡 重點摘要

- **事件在資料變更成功後發送**（「先寫入，再廣播」），避免接收端處理時資料尚不存在。
- `axios.post` 是非同步的——`await` 確保事件發送完成後才回傳 HTTP 回應給客戶端。
- 目前 Event Bus 對所有 Service 進行無條件廣播，這是教學用途的簡化設計。
- 在 Service 尚未實作事件接收端前，Event Bus 對它們的請求會回傳 404——這不是錯誤，是正常的開發過渡期現象。

## 🔑 關鍵字

`PostCreated`, `CommentCreated`, `Event Emission`, `Axios`, `Broadcasting`
