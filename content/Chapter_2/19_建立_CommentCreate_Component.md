# 建立 CommentCreate Component

## 📝 課程概述

本單元實作了 `CommentCreate` 元件，讓使用者在每篇文章下方填入留言內容並送出。這個 Component 接收 `postId` 作為 prop（由父層 `PostList` 傳入），決定這則留言要附加到哪篇文章。送出成功後，同時清空輸入框作為 UX 反馈。

## 核心觀念與實作解析

### Props 設計：從父層取得 `postId`

留言必然附屬於某篇文章，因此 `CommentCreate` 元件**無法獨立存在**——它必須知道要向哪個 `postId` 附加留言。

```jsx
// PostList 中的調用方式
<PostList posts={posts} />
// PostList 內部 map posts:
// {posts.map(post => (
//   <div key={post.id}>
//     <h3>{post.title}</h3>
//     <CommentCreate postId={post.id} />  {/* 父層傳入 postId */}
//     <CommentList postId={post.id} />
//   </div>
// ))}
```

### CommentCreate Component 實作

```jsx
const CommentCreate = ({ postId }) => {
  const [content, setContent] = useState('');

  const onSubmit = async (e) => {
    e.preventDefault();
    await axios.post(`http://localhost:4001/posts/${postId}/comments`, {
      content
    });
    setContent('');
  };

  return (
    <form onSubmit={onSubmit}>
      <div className="form-group">
        <label>New Comment</label>
        <input
          className="form-control"
          value={content}
          onChange={e => setContent(e.target.value)}
        />
      </div>
      <button className="btn btn-primary">Submit</button>
    </form>
  );
};
```

### 為什麼成功後要 `setContent('')`？

這是一個微小的 UX 設計選擇：用 `setContent('')` 清空輸入框，讓使用者清楚知道「請求已送出，系統已接收」。如果沒有這步，使用者可能會以為點擊沒反應而重複點擊 Submit。

### POST URL 的意義

`http://localhost:4001/posts/${postId}/comments` 對應 Comments Service 的端點設計：
- `POST /posts/:id/comments`：對 ID 為 `:id` 的文章新增留言，request body 攜帶 `content` 欄位。

## 💡 重點摘要

- `CommentCreate` 是**哑元件（Dumb Component）**，只負責呈現表單與發送請求，不管理任何資料狀態。
- **`postId` 必須從父層 Prop 傳入**——元件無法自己猜測要附屬到哪篇文章。
- 成功後 `setContent('')` 是 UX 細節，用視覺回饋確認請求已送出。
- API URL 的設計體現了 RESTful 的「資源導向」原則：`/posts/:id/comments` 即「某文章下的留言」。

## 🔑 關鍵字

`CommentCreate`, `Props`, `Dumb Component`, `RESTful URL`, `Controlled Component`
