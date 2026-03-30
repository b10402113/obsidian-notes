# Query Service 實作：事件處理邏輯與 React 前端整合

## 📝 課程概述

本單元實作 Query Service 的核心功能——**事件處理邏輯**。Query Service 根據收到的 `type`（事件類型）來決定如何更新自己的資料結構：收到 `PostCreated` 就新增文章；收到 `CommentCreated` 就把留言附加到對應文章的陣列中。實作完成後，我們將 React App 改為只向 Query Service 發送一次請求，完美消滅了 N+1 問題。

---

## 核心觀念與實作解析

### Query Service 的資料結構

```javascript
const posts = {};
// 結構：
// {
//   "post-id-1": {
//     id: "post-id-1",
//     title: "New Post",
//     comments: [
//       { id: "comment-id-1", content: "Great post!", status: "pending" }
//     ]
//   }
// }
```

每一篇文章都是一個 key，值是包含 `id`、`title`、`comments` 的物件。`comments` 預設為空陣列，等到收到 `CommentCreated` 事件時再動態填入。

---

### 事件處理：POST /events 端點實作

```javascript
app.post('/events', (req, res) => {
  const { type, data } = req.body;

  if (type === 'PostCreated') {
    posts[data.id] = { id: data.id, title: data.title, comments: [] };
  }

  if (type === 'CommentCreated') {
    const { id, content, postId, status } = data;
    posts[postId].comments.push({ id, content, status });
  }

  console.log(posts); // 除錯用
  res.send({});
});
```

> 這裡預留了 `status` 欄位，是為了即將到來的 Moderation（審核）功能做準備。

---

### React App 重構：從 N 個請求到 1 個請求

**修改前的架構（PostList）：**
```javascript
// 向 Post Service 要文章
const res = await axios.get('http://localhost:4000/posts');

// 對每篇文章，向 Comment Service 請求留言 → N+1 問題！
```

**修改後的架構（PostList）：**
```javascript
// 只向 Query Service 發送一次請求
const res = await axios.get('http://localhost:4002/posts');
setPosts(res.data); // 直接設定 state
```

每篇文章的 `comments` 已經內嵌在 post 物件中，**不再需要**單獨呼叫 Comment Service。留言資料由 Query Service 統一維護，React App 只需要對 `localhost:4002` 發送一次請求。

---

### ⚡ 實驗：關閉依賴 Service，Query Service 仍能正常運作

這是最能體現事件驅動架構價值的實驗環節。當我們：

1. **關閉 Post Service**（停止進程）
2. **關閉 Comment Service**（停止進程）
3. 重新整理 React 頁面

結果：**Query Service 仍然正常回傳所有資料**。

這是因為 Query Service 已經收到了所有歷史事件並保存在記憶體中。即便上游的 Production Service 全數崩潰，Query Service 的查詢功能不受任何影響——**這就是 Zero Dependency 的威力**。

當然，若嘗試**新增文章或留言**，就會失敗（因為 Production Service 已關閉），但**查詢功能完全正常**。

---

## 💡 重點摘要

- Query Service 的 `GET /posts` 是 React App 的**單一真相來源（Single Source of Truth）**，從此不再需要對每個 Service 發送獨立請求。
- 事件處理邏輯的核心是：**根據 event.type 執行對應的資料寫入操作**。
- `status: 'pending'` 欄位的預留，是為 Moderation 功能做的前期準備。
- 事件驅動架構的最大價值在此刻展現：**查詢功能完全與 Production Service 解耦**。

## 關鍵字

Query Service Event Handler, PostCreated, CommentCreated, Event-Driven Data, Single Source of Truth, N+1 Solution, React Integration, Zero Dependency, In-Memory State, Query Service GET /posts
