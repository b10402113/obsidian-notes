# React 專案設定

## 📝 課程概述

本單元建立了 React 前端應用（`client`，Port 3000），並規劃了整個 Component 層級架構。我們從零開始清理 Create React App 的預設樣板，只保留必要的 `index.js` 與 `App.js`，並加入 Bootstrap 來快速提升 UI 可讀性。

## 核心觀念與實作解析

### 為什麼要自己刻 Component？

許多同類課程會直接提供完整 React 程式碼。本課程選擇讓學員親手建立，是因為後續章節中 React 程式碼會有多次變動（連線到不同後端 Service），自己動手刻能確保理解每個改動背後的意義。

### Component 層級架構

```
App
├── PostCreate         （頂端表單：建立文章）
└── PostList
    └── [每篇文章的卡片]
        ├── <h3> 標題
        ├── CommentCreate  （留言表單）
        └── CommentList   （留言列表）
```

### 檔案結構

```
client/src/
├── index.js     — ReactDOM.render(<App />, document.getElementById('root'))
└── App.js       — 主 Component，組裝所有子元件
```

### 加入 Bootstrap（僅 CSS）

在 `client/public/index.html` 的 `<head>` 中加入：

```html
<link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css">
```

老師特別刪除了 Bootstrap 的 JavaScript CDN 連結——我們只需要 CSS，不需要 Bootstrap 的 JS 互動功能。

### PostCreate Component 實作重點

```jsx
const [title, setTitle] = useState('');

const onSubmit = async (e) => {
  e.preventDefault();
  await axios.post('http://localhost:4000/posts', { title });
  setTitle('');
};
```

- 使用 **Controlled Component**（`value={title}` + `onChange`）來追蹤輸入框狀態。
- 使用 **async/await** 處理 API 請求，比 `.then()`鏈更直覺。
- 提交成功後將 `title` 重設為空字串，作為 UX 上的成功回饋。
- 送出請求的 URL 是 `localhost:4000/posts`——這是後端 Posts Service 的地址。

### 初始測試與預期錯誤

第一次 Submit 時，瀏覽器 Console 會出現 **`CORS 錯誤`**（`Access-Control-Allow-Origin`）。這是因為瀏覽器從 Port 3000（React）發送請求到 Port 4000（Posts Service），屬於**跨來源（Cross-Origin）請求**——這是下個單元要處理的議題。

## 💡 重點摘要

- **Controlled Component** 模式：`useState` + `value` + `onChange` 三件套缺一不可。
- **Bootstrap CSS** 可透過 CDN 快速引用，無需 `npm install`。
- React app 的 **CORS 錯誤**不是 React 端的問題，而是後端 Server 缺少設定——修復在後端。
- 這個 React 程式碼會隨課程進展多次修改，每次修改都對應了不同的微服務架構選擇。

## 🔑 關鍵字

`Create React App`, `Controlled Component`, `Bootstrap`, `CORS`, `async/await`
