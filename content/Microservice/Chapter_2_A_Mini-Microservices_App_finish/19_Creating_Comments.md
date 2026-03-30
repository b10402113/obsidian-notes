# Comment Create 與 Comment List 元件：串接後端留言 API

## 📝 課程概述

本單元完成 React 前端最後兩個元件——`CommentCreate` 與 `CommentList`。這是整個前端實作的最後一塊拼圖。完成後，整個應用程式（Post Service + Comment Service + React App）的 MVP 就完整了。我們會發現一個明顯的效能問題：每次渲染頁面時，CommentList 會對每篇文章各自獨立地發送一次 HTTP 請求到 Comment Service，導致 N 篇文章就產生 N 次 API 呼叫。

---

## 核心觀念與實作解析

### CommentCreate 元件：接收 Post ID 作為 Props

CommentCreate 是一個表單元件，用來對特定文章新增留言。它**必須**知道自己在為哪篇文章新增留言——這個資訊由父層元件（PostList）透過 props 傳入。

```jsx
const CommentCreate = ({ postId }) => {
  const [content, setContent] = useState('');

  const onSubmit = async (e) => {
    e.preventDefault();
    await axios.post(`http://localhost:4001/posts/${postId}/comments`, { content });
    setContent('');
  };

  return (
    <form onSubmit={onSubmit}>
      <input value={content} onChange={(e) => setContent(e.target.value)} />
      <button type="submit">Submit</button>
    </form>
  );
};
```

### CommentList 元件：初始化空陣列，fetch 單篇文章的留言

CommentList 與 PostList 的資料格式不同——Comment Service 的 GET `/posts/:id/comments` 回傳的是**陣列**，而非物件：

```jsx
const [comments, setComments] = useState([]); // 注意是空陣列，不是 {}

useEffect(() => {
  const fetchData = async () => {
    const res = await axios.get(`http://localhost:4001/posts/${postId}/comments`);
    setComments(res.data);
  };
  fetchData();
}, [postId]);
```

### Props 穿透（Prop Drilling）：PostList → CommentList

PostList 在 map 渲染每篇文章時，同時 render `CommentCreate` 與 `CommentList`，並透過 props 將 `post.id` 傳遞下去：

```jsx
{posts.map(post => (
  <div key={post.id}>
    <h3>{post.title}</h3>
    <CommentList postId={post.id} />
    <CommentCreate postId={post.id} />
  </div>
))}
```

---

### 🚨 效能瓶頸：N+1 Request Problem

當頁面渲染完成後，打開瀏覽器的 Network 面板，你會看到這樣的景象：每篇文章都觸發一次對 Comment Service 的 GET 請求。若有 3 篇文章，就會有 3 次獨立的 HTTP 請求。

這稱為 **N+1 Request Problem**（N+1 請求問題）：取得 N 筆資料需要 N 次額外的附屬請求。

> 這在 microservices 架構下尤其成為問題，因為 Comment Service 和 Post Service 是分開的，無法像 Monolithic 架構那樣在一次資料庫查詢中完成 JOIN。

這個問題預示著我們即將進入下一個階段：**引入 Query Service 與 Event Bus，透過事件驅動機制來消滅這個 N+1 問題**。

---

## 💡 重點摘要

- CommentCreate 和 CommentList 都需要接收 `postId` prop，才能知道自己在操作哪篇文章。
- Comment Service 的 GET 回傳**陣列**，因此 state 初始化為 `[]`；Post Service 的 GET 回傳**物件**，state 初始化為 `{}`。
- 目前 CommentList 元件在頁面載入時**不會自動刷新**——新增留言後必須手動刷新頁面才能看到新留言。
- N+1 Request Problem 是這個 MVP 版本的最大效能瓶頸，解決方案是 Query Service。

## 關鍵字

N+1 Request Problem, CommentCreate, CommentList, Prop Drilling, Props, POST /posts/:id/comments, GET /posts/:id/comments, In-Memory State, useEffect, React Component Design
