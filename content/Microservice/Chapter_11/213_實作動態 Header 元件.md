# 實作動態 Header 元件

## 📝 課程概述

本單元實作 **Header 元件**（頂部導航列），它會根據使用者的登入狀態動態切換顯示內容：**未登入時顯示 Sign Up 與 Sign In 連結；已登入時顯示 Sign Out 連結。** Header 放在 `_app.js` 中渲染，確保每個頁面都有一致的導航體驗。

## 核心觀念與實作解析

### 使用 Next.js 的 `Link` 元件

Next.js 不使用傳統的 `<a href="...">` 標籤做客戶端導航，而是用 `<Link>` 包住 `<a>`：

```jsx
import Link from 'next/link';

<Link href="/auth/signup">
  <a className="nav-link">Sign Up</a>
</Link>
```

`Link` 負責攔截點擊事件，透過 Next.js 內部的客戶端路由進行無刷新切頁；`<a>` 標籤則負責實際的視覺呈現。

### 根據 `currentUser` 動態切換內容

```jsx
function Header({ currentUser }) {
  const links = [
    !currentUser && { label: 'Sign Up', href: '/auth/signup' },
    !currentUser && { label: 'Sign In', href: '/auth/signin' },
    currentUser && { label: 'Sign Out', href: '/auth/signout' },
  ]
    .filter(Boolean)
    .map(({ label, href }) => (
      <li key={href}>
        <Link href={href}>
          <a className="nav-link">{label}</a>
        </Link>
      </li>
    ));

  return (
    <nav className="navbar navbar-light bg-light">
      <div className="d-flex justify-content-end">
        <ul className="nav">{links}</ul>
      </div>
    </nav>
  );
}
```

### 這個技巧的亮點

透過 **`!currentUser && {...}` 產生 `false` 或物件**，再 `.filter(Boolean)` 移除 `false`，最後 `.map()` 渲染——可以在**一行表達式**內完成「根據條件動態增減清單項目」的邏輯，完全不需要 `if/else`。

### Header 整合進 `_app.js`

```jsx
// _app.js
import Header from '../components/Header';

export default function App({ Component, pageProps, currentUser }) {
  return (
    <>
      <Header currentUser={currentUser} />
      <Component {...pageProps} />
    </>
  );
}
```

Header 元件現在從 `_app.js` 接收 `currentUser`，因此**每個頁面都能看到正確的登入狀態**。

---

## 💡 重點摘要

- **`Link` 是 Next.js 的客戶端路由元件，須包住 `<a>` 標籤才能正常運作。**
- **`.filter(Boolean).map()` 鏈式操作能在單一表達式內完成「條件性渲染清單項目」，不需要額外的 control flow。**
- **Header 放在 `_app.js` 渲染，能確保所有頁面都共享同一份 Header，且能從 Props 拿到 `currentUser`。**

## 🔑 關鍵字

Link, Header, currentUser, filter(Boolean), 動態清單, _app.js, 客戶端路由
