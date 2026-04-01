# JWT 與 Server-Side Rendering 的衝突：為何需要 Cookie

## 📝 課程概述

本單元分析了我們正在建構的 **Server-Side Rendering（SSR）** 應用程式與 JWT 傳輸方式之間的張力。課程結論是：由於 SSR 的第一個請求**無法透過 JavaScript 自訂**，我們**必須使用 Cookie 來攜帶 JWT**。理解這個「為什麼」，比記住「用 Cookie」這個結論更重要。

---

## 核心觀念與實作解析

### 普通 React App 的請求時序

```
1. 輸入 URL → 取得 HTML 檔案（幾乎無內容）
2. 瀏覽器解析 script tags → 發出請求取得 JS bundles
3. React 啟動 → 發出 API 請求取得資料
4. 渲染畫面
```

在這個流程中，**最早需要驗證資訊的時機是第 3 步**（React 啟動後的 API 請求）。此時我們可以：
- 在 `Authorization` header 中手動加入 JWT
- 將 JWT 放在 request body 中
- 使用 Cookie

三種方式都可以，因為我們可以在 React 程式碼中**主動控制**這些請求。

---

### Server-Side Rendering 的請求時序

```
1. 輸入 URL → Next.js Server 直接發回「已完整渲染的 HTML」（含所有內容）
```

這裡的關鍵差異：**第一個 HTTP 請求不是由 JavaScript 發出的，而是由瀏覽器本身直接發出的**。當使用者在網址列輸入 URL 並按下 Enter 時，瀏覽器**無法執行任何 JavaScript** 來自訂這個請求。

因此：
- **無法**在第一個請求中加入 `Authorization` header（瀏覽器不允許）
- **無法**將 JWT 放在 body 中（GET 請求通常沒有 body）
- **唯一可行**：讓瀏覽器**自動帶上**驗證資訊——這正是 **Cookie 的原生行為**

---

### Cookie 的自動攜帶特性

Cookie 的設計就是為了解決這個問題：當瀏覽器收到 `Set-Cookie` 回應後，**未來對同一個 domain 的所有請求，瀏覽器會自動附加這個 Cookie**。這包括 SSR 的第一個請求，開發者無需寫任何 JavaScript 來控制它。

> **結論**：Cookie 是**瀏覽器層級的自動機制**，而 JavaScript 控制適用於「React 啟動後的 API 請求」。

---

## 💡 重點摘要

- **SSR 應用程式的第一個請求無法由 JavaScript 自訂，唯一能攜帶驗證資訊的途徑是 Cookie。**
- **Cookie 與 JWT 的組合是：JWT 負責「驗證內容與防篡改」，Cookie 負責「自動傳輸 JWT」。**
- **Server-Side Rendering 的 SEO 與首頁載入速度優勢，換來了這個驗證架構上的限制——這是一個有意識的取捨。**

---

## 🔑 關鍵字

Server-Side Rendering, Next.js, Cookie, Authorization Header, JWT, First Request
