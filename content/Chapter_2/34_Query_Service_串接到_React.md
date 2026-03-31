# Query Service 串接到 React

## 📝 課程概述

本單元將 React 前端從原本的「多次請求模式」（分別打 Posts Service 和 Comments Service）改為「單次請求模式」——只向 Query Service 的 `GET /posts` 端點發送一次請求，取得包含所有文章與留言的完整嵌套資料結構。同時也簡化了 `CommentList` 元件，不再自行發送請求，改為直接使用父層 Prop 傳入的留言資料。

## 核心觀念與實作解析

### PostList 的修改：從 Query Service 取資料

```jsx
// PostList.js
useEffect(() => {
  const fetchPosts = async () => {
    const res = await axios.get('http://localhost:4002/posts');
    setPosts(res.data);
  };
  fetchPosts();
}, []);
```

URL 從 `localhost:4000/posts`（Posts Service）改為 `localhost:4002/posts`（Query Service）。

### 回傳資料的結構

```json
{
  "post-id-1": {
    "id": "post-id-1",
    "title": "My Post",
    "comments": [
      { "id": "c1", "content": "Great!" }
    ]
  }
}
```

### CommentList 的簡化：不再自己發請求

原本的 `CommentList` 會自己向 Comments Service 發請求抓留言。現在既然留言已經在 `PostList` 取得時就嵌入在 Post 物件中，`CommentList` 只需要接收 Prop：

```jsx
// CommentList.js — 修改後
const CommentList = ({ comments }) => {
  const renderedComments = comments.map(comment => (
    <li key={comment.id}>{comment.content}</li>
  ));
  return <ul>{renderedComments}</ul>;
};
```

### PostList 中的 Prop 傳遞

```jsx
// PostList.js
{Object.values(posts).map(post => (
  <div key={post.id}>
    <h3>{post.title}</h3>
    <CommentCreate postId={post.id} />
    <CommentList comments={post.comments} /> {/* 直接傳入留言陣列 */}
  </div>
))}
```

### 驗證 Event-Driven 架構的威力

最戲劇性的測試：**直接 kill 掉 Posts Service 和 Comments Service**，只保留 Query Service 運行。

- **結果**：前端仍能正常顯示所有文章與留言，**使用者的讀取體驗完全不受影響**！
- 嘗試建立新文章/留言會失敗（因為對應的 Service 掛了），但閱讀功能完全正常。

這正是「服務獨立性」最有力的證明。

## 💡 重點摘要

- **一次請求取代多次請求**：`GET /posts` 到 Query Service 直接取得完整嵌套資料，消滅 N+1 問題。
- **Prop Drilling**：`CommentList` 從自行發請求改為接收 `comments` prop，React 資料流向更清晰。
- **Query Service 獨立運行測試**：kill 掉 Posts 和 Comments 後，Query Service 仍能持續服務讀取請求——驗證了 Event-Driven 架構的韌性。
- 前端只需改一個 URL（從 4000 改到 4002），其餘 React 程式碼幾乎不變——這是「去耦合」帶來的維護便利性。

## 🔑 關鍵字

`PostList Refactor`, `Prop Drilling`, `Single Request`, `Service Independence`, `Event-Driven Resilience`
