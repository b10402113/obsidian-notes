# 建立 CommentList Component

## 📝 課程概述

本單元實作了 `CommentList` 元件，負責向 Comments Service 請求並渲染指定文章的留言清單。這是最後一個 React 元件——完成後，應用程式的基本 CRUD 功能已全部串接完成，但同時也即將暴露出**微服務架構的根本效能問題**。

## 核心觀念與實作解析

### CommentList 的資料需求

`CommentList` 需要知道要取得哪篇文章的留言，因此同樣從父層接收 `postId` prop，並據此發送對應的 API 請求。

```jsx
const CommentList = ({ postId }) => {
  const [comments, setComments] = useState([]);

  useEffect(() => {
    const fetchData = async () => {
      const res = await axios.get(`http://localhost:4001/posts/${postId}/comments`);
      setComments(res.data);
    };
    fetchData();
  }, [postId]);

  const renderedComments = comments.map(comment => (
    <li key={comment.id}>{comment.content}</li>
  ));

  return <ul>{renderedComments}</ul>;
};
```

### 為什麼使用 `useEffect` 並傳入 `[postId]`？

`useEffect` 的第二個參數陣列用來控制 Effect 的觸發時機：
- **空陣列 `[]`**：Effect 只在 Component **首次渲染**時執行一次（適合從後端抓資料的場景）。
- **`[postId]`**：每當 `postId` 改變時，重新執行 Effect（即對新的文章 ID 重新抓取留言）。

當我們在 `PostList` 中 render 多個 `CommentList`（每篇文章一個），每個 `CommentList` 都是獨立的 Component，擁有各自獨立的 `useEffect`，各自只抓取自己文章的留言。

### N+1 問題的浮現

**這裡就是微服務架構的第一個大問題。**

假設頁面有 3 篇文章：
1. React 向 `localhost:4000/posts` 發送 GET → 取得 3 篇文章
2. 對第 1 篇文章：React 向 `localhost:4001/posts/post1/comments` 發送 GET
3. 對第 2 篇文章：React 向 `localhost:4001/posts/post2/comments` 發送 GET
4. 對第 3 篇文章：React 向 `localhost:4001/posts/post3/comments` 發送 GET

**總共發送了 4 次請求**——1 次抓文章，3 次分別抓各篇文章的留言。這是經典的 **N+1 問題（N+1 Query Problem）**。

> 在 Monolith（單體式）架構中，一個端點 `GET /posts?includeComments=true` 就能一次取回所有文章與留言；但在微服務架構下，這個「簡單需求」變成了跨服務資料聚合的挑戰。

## 💡 重點摘要

- `CommentList` 是**自主管理資料的元件**，使用 `useEffect` 在初始渲染時抓取留言。
- **`useEffect` 的第二個參數**控制何時重新執行 Effect——這是 React Hooks 的核心概念。
- **N+1 問題**：3 篇文章觸發 4 次網路請求（N 篇文章需要 N+1 次請求）。
- 這個效能問題正是後續章節引進「非同步通訊」與「Query Service」的契機。

## 🔑 關鍵字

`CommentList`, `useEffect`, `N+1 Problem`, `Cross-Service Query`, `React Hooks`
