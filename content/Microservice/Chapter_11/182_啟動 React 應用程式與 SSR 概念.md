# 啟動 React 應用程式與 SSR 概念

## 📝 課程概述

本單元拉開了 React 前端開發的序幕，我們將為既有 Auth Service 建構一個具備 **Server-Side Rendering（SSR）** 能力的 React 應用程式。老師先說明本階段的目標：僅實作與 Authentication 相關的頁面（登入、註冊、登出），並預告整個專案將分階段完成。

## 核心觀念與實作解析

### 傳統 Client-Side Rendering 的請求流程

在傳統 React 開發模式下，使用者造訪 URL 時，瀏覽器至少發出 2~3 次請求：

1. 第一次請求取得幾乎為空的 HTML 骨架（含 `<script>` 標籤）
2. 瀏覽器下載並執行 JavaScript，React 啟動
3. React 組件才開始發送 API 請求（如 `GET /api/orders`）取得資料後再 render

這導致使用者看到第一個有意義畫面的時間較長，且不利於 SEO。

### Server-Side Rendering 的核心差異

Next.js 實現 SSR 的方式是：

1. 瀏覽器發送單一請求至 Next.js 開發伺服器
2. Next.js 在**伺服器端**就向各 Service 發送請求抓取資料
3. Next.js 將**已經帶有完整內容的 HTML** 一次性回傳給瀏覽器

> 如此一來，使用者在第一次請求時就能看到完整內容，行動裝置載入速度更快，且搜尋引擎更容易抓取內容。

### 我們即將建構的頁面藍圖

本階段我們僅實作 Authentication 相關功能：

| 頁面 | 路徑 | 說明 |
|------|------|------|
| Landing Page | `/` | 根據登入狀態顯示「你已登入」或「你未登入」 |
| 登入頁 | `/auth/signin` | Email + Password 表單 |
| 註冊頁 | `/auth/signup` | Email + Password 表單 |
| Header | 通用頂部導航列 | 依登入狀態動態切換連結 |

### 教學結語：React 非必修

老師特別說明，如果你對 React 完全不熟悉，或對前端不感興趣，課程尾聲有提供一個 zip 檔，內含預先建好的 React 程式碼，且須配合閱讀文字說明修改 Kubernetes 部署設定。**但強烈建議自己動手做**，才能理解 SSR 與微服務整合的種種眉角。

---

## 💡 重點摘要

- **傳統 CSR 需要 2~3 次請求才能看到內容；SSR 在伺服器端預先抓取資料，單次請求就能回傳完整 HTML。**
- **Next.js 是實現 SSR 最便利的框架，能大幅簡化本應有的繁瑣設定。**
- **本階段目標是 Auth 相關頁面：Landing Page、Sign In、Sign Up，以及一個動態 Header。**
- **即使不熟悉 React，動手實作能幫助你理解 SSR 與微服務整合的深層議題。**

## 🔑 關鍵字

Server-Side Rendering, Next.js, CSR, Landing Page, Auth
