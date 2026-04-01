# 實作 Sign In 表單

## 📝 課程概述

本單元實作 Sign In（登入）頁面。由於它與 Sign Up 頁面結構幾乎完全相同——都是 Email + Password 表單，老師選擇**直接複製 `signup.js` 並修改兩處**（URL 與按鈕文字），而非建立共用的表單 Component，以保持本課程對微服務主題的專注。

## 核心觀念與實作解析

### 與 Sign Up 的差異

| 差異點 | Sign Up | Sign In |
|--------|---------|---------|
| API URL | `/api/users/signup` | `/api/users/signin` |
| 按鈕文字 | Sign Up | Sign In |
| H1 標題 | Sign Up | Sign In |

### 快速複製流程

```
pages/auth/signup.js  →  複製為  →  pages/auth/signin.js
```

在 `signin.js` 中搜尋替換：
1. URL：`/api/users/signup` → `/api/users/signin`
2. 按鈕文字：`Sign Up` → `Sign In`

### 測試流程

1. 先清除瀏覽器的 Cookie（確保為未登入狀態）
2. 前往 `/auth/signin`
3. 填入曾經註冊過的 Email 與密碼
4. 點擊 Sign In
5. 確認：收到 `Set-Cookie`、成功跳轉至 Landing Page、Header 顯示「Sign Out」

> 如果忘記之前註冊過的帳號密碼，重新跑一次 Sign Up 註冊新帳號即可。

---

## 💡 重點摘要

- **Sign In 與 Sign Up 的實作差異極小，僅需修改 API URL 與表單文字。**
- **複製後替換是快速建立相似頁面的合理策略；未來如果 React 是主要焦點，可以進一步抽取成共用的表單 Component。**
- **Sign In 成功後 Auth Service 同樣會回傳 `Set-Cookie`，瀏覽器自動管理 session。**

## 🔑 關鍵字

Sign In, /api/users/signin, Set-Cookie, 表單複製, session Cookie
