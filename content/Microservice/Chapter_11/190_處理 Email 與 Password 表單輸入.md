# 處理 Email 與 Password 表單輸入

## 📝 課程概述

本單元實作 Sign Up 表單的前端程式碼，包括：**表單 UI 搭建**、**React `useState` 追蹤輸入狀態**，以及**串接 Axios 向 Auth Service 發出 `POST /api/users/signup` 請求**。我們將在此確認整個 Client → Ingress NGINX → Auth Service 的請求鏈路是否暢通。

## 核心觀念與實作解析

### 表單 UI（使用 Bootstrap Classes）

```jsx
<div className="form-group">
  <label>Email Address</label>
  <input type="email" className="form-control" />
</div>
<div className="form-group">
  <label>Password</label>
  <input type="password" className="form-control" />
</div>
<button className="btn btn-primary">Sign Up</button>
```

### React `useState` 追蹤表單資料

```javascript
const [email, setEmail] = useState('');
const [password, setPassword] = useState('');
```

將 state 與 input 的 `value` 及 `onChange` 綁定：

```jsx
<input
  value={email}
  onChange={(e) => setEmail(e.target.value)}
  className="form-control"
/>
```

### `onSubmit` 處理與 `e.preventDefault()`

表單預設的提交行為是瀏覽器直接發送 `GET/POST` 請求並重載頁面。我們需要攔截這個行為，改由 JavaScript 控制：

```javascript
const onSubmit = async (event) => {
  event.preventDefault();
  // 使用 Axios 發送請求
};
```

### 向 Auth Service 發送請求

使用 Axios 發送 `POST` 請求：

```javascript
const response = await axios.post('/api/users/signup', {
  email,
  password,
});
```

成功後，Auth Service 會在回應的 `Set-Cookie` Header 中寫入 session cookie，瀏覽器會自動管理這個 cookie，日後所有後續請求都會自動帶上。

### 請求路徑的完整流向

```
瀏覽器（axios.post('/api/users/signup')）
  → Ingress NGINX（ticketing.dev:80）
    → ClusterIP Service（auth-svc:3000）
      → Express Router → 最終處理函式
```

> 這條路徑上每一層的轉發都已在前面章節設定完成，此刻驗證的正是這整條鏈路的正確性。

---

## 💡 重點摘要

- **`e.preventDefault()` 是表單處理的第一步，阻止瀏覽器預設的提交行為與頁面重載。**
- **`useState` 是 React 中追蹤表單資料的標準方式，將每個 input 的值與 state 雙向綁定。**
- **瀏覽器收到 `Set-Cookie` Header 後會自動儲存，日後所有請求自動攜帶，無需手動管理。**
- **`POST /api/users/signup` 的整條請求鏈路（Client → Ingress NGINX → Auth Service）在此驗證暢通。**

## 🔑 關鍵字

useState, onSubmit, preventDefault, Axios, Set-Cookie, Form Group
