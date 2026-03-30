# React 前端設定：從零打造 Blog 應用程式 UI

## 📝 課程概述

本單元建立 React 前端應用程式，作為整個 microservices 系統的 Client。我們會從 Create React App 出發，逐步建構出 Post Create、Post List、Comment List 與 Comment Create 四個核心元件，並整合 Bootstrap 來快速提升 UI 品質。重點在於讓 React App 能向多個獨立的後端 Service 發送 HTTP 請求。

---

## 核心觀念與實作解析

### 專案架構回顧

```
[React App :3000] → HTTP → [Post Service :4000]
                              [Comment Service :4001]
```

React App 同時需要與 Post Service 和 Comment Service 通訊，這種架構在未來會暴露出 CORS 與跨 Service 資料同步的問題。

---

### 元件層級架構（Component Hierarchy）

```
App
├── PostCreate         // 頂部表單：用來建立文章
└── PostList
    ├── Post (重複渲染)
    │   ├── CommentList    // 該文章的留言列表
    │   └── CommentCreate  // 該文章的留言輸入框
    └── ...
```

每一篇 Post 底下都有屬於自己的 CommentList 與 CommentCreate 元件，這樣每篇文章的留言系統是完全獨立的。

---

### React 程式碼實作重點

#### 1. 刪除預設 boilerplate，從乾淨的起點開始

Create React App 預設會產生大量我們不需要的程式碼。直接刪除 `src/` 下的所有檔案，從頭建立 `index.js` 與 `App.js` 是最快的做法。

#### 2. PostCreate 元件：雙向資料繫結 + Async Request

```javascript
const [title, setTitle] = useState('');

const onSubmit = async (e) => {
  e.preventDefault();
  await axios.post('http://localhost:4000/posts', { title });
  setTitle(''); // 清空輸入框，表示請求成功
};
```

- 使用 `e.preventDefault()` 阻止表單預設的頁面跳轉行為
- `async/await` 語法讓非同步請求的處理更直觀
- 成功後將 `title` 重設為空字串，是一種 UX 暗示（告知使用者請求已完成）

#### 3. PostList 元件：useEffect + 初始化 State

```javascript
const [posts, setPosts] = useState({});

useEffect(() => {
  fetchPosts();
}, []); // 空陣列 = 元件首次渲染時執行一次

const fetchPosts = async () => {
  const res = await axios.get('http://localhost:4000/posts');
  setPosts(res.data);
};
```

> 特別注意：`posts` 的初始值是 `{}`（空物件），而非 `[]`（空陣列）。這是因為 Post Service 的 GET `/posts` 回傳的是 `{ [id]: { id, title } }` 格式的物件，而非陣列。

#### 4. 渲染 Post 清單

```javascript
const renderedPosts = Object.values(posts).map(post => (
  <div key={post.id} className="card">
    <h3>{post.title}</h3>
  </div>
));
```

`Object.values(posts)` 將物件轉為陣列，以便用 `.map()` 逐一渲染。每個元素的 `key` 使用 `post.id` 來確保 React 能正確追蹤列表元素。

#### 5. Bootstrap 整合

在 `public/index.html` 的 `<head>` 中加入 Bootstrap CDN link tag（注意：只需要 CSS link，**不需要** Bootstrap 的 JS 檔案，因為我們不需要任何 Bootstrap 的互動元件）。這讓所有元件在加入 `className` 後能自動獲得 Bootstrap 的樣式。

---

## 💡 重點摘要

- React App 同時作為多個後端 Service 的 client，這是 microservices 前端的典型架構。
- `useEffect` 的第二個參數使用空陣列，代表這個 effect 只在元件首次掛載（mount）時執行一次。
- Post Service 的資料格式是物件（`{ id: { id, title } }`），React 端必須用 `Object.values()` 轉為陣列才能 map 渲染。
- Bootstrap 在這個階段只用來做靜態樣式，不需要引入其 JS 檔案。

## 關鍵字

Create React App, Component Hierarchy, PostCreate, PostList, useState, useEffect, Object.values, async/await, Bootstrap Integration, HTTP Request, React Frontend
