# Micro-frontends 架構模式

## 📝 課程概述

本單元補完了微服務架構的最後一塊拼圖——**前端（Frontend）**。當後端已經遷移到微服務並享受獨立團隊、獨立部署的好處時，許多組織的前端仍然是一個巨大的 Monolith，成為整個系統的瓶頸。Micro-frontends 正是用來解決這個問題的架構模式：將前端的單一程式碼庫拆分為多個獨立運行的小型單頁應用程式（Single Page Application），由不同團隊各自維護與部署。

## 核心觀念與實作解析

### Monolithic Frontend 的問題

想像一個線上學習平台，後端已遵循微服務最佳實踐拆分——Course Discovery Service、Course Recommendations Service、User Service、Enrollment Service 等各自獨立。但前端呢？

**通常前端仍然是一個 Monolith**：
- 整個網站的 HTML/CSS/JS 在同一個 Codebase 中
- 所有功能由一個前端團隊維護
- 整個應用程式從 Web Application Service 一次性發送到使用者瀏覽器

這帶來了兩個層面的問題：

#### 組織層面：單一前端團隊成為瓶頸

- Course Discovery 團隊要新增功能 → 必須請求前端團隊在 UI 上實作 → **等待與依賴**
- Course Recommendations 團隊要變更顯示的資料 → 同樣需要前端團隊配合
- 前端團隊要改進 User Profile 頁面 → 必須學習並整合 User Service 的 API
- 前端團隊要優化 Enrollment 流程 → 必須熟悉 Enrollment Service 與 Payment Service 的商業邏輯

> **結論**：一個前端團隊被迫成為所有後端微服務團隊的共同依賴點——這正是微服務架構試圖消除的瓶頸，只不過現在出現在前端。

#### 技術層面：大型 Codebase 的維護噩夢

- 隨著業務成長，程式碼庫變得龐大且難以理解
- 測試時間大幅拉長
- 任何一個 UI 功能的變更，都需要**整個前端程式碼重新建置、重新測試、重新部署**

---

### Micro-frontends 模式的核心思想

Micro-frontends 將 Monolithic Web Application 拆分為多個**獨立的單頁應用程式**，每個對應一個業務功能領域：

```
┌─────────────────────────────────────────────────────┐
│                   Container Application               │
│         (Header/Footer + Auth + Shared Libs)         │
└─────────────────────────────────────────────────────┘
  ↑           ↑           ↑           ↑
  ↓           ↓           ↓           ↓
┌──────┐  ┌────────┐  ┌─────────┐  ┌──────────┐
│Course│  │Course  │  │  User   │  │Enrollment│
│Discov-│  │Recomm. │  │ Profile │  │  Flow    │
│ ery   │  │        │  │         │  │          │
│Micro- │  │Micro-  │  │Micro-   │  │Micro-    │
│frontend│ │frontend│  │frontend │  │frontend  │
└──────┘  └────────┘  └─────────┘  └──────────┘
```

**關鍵特性**：

- 每個 Micro-frontend **完全解耦**，知道如何自行載入、mount 到 DOM 與 unmount。
- 每個 Micro-frontend 可以作為獨立 Web App 在瀏覽器中單獨測試。
- **每個 Micro-frontend 由一個獨立的全端團隊擁有**，具備完整的技術能力與領域知識。
- 所有 Micro-frontends 在**使用者瀏覽器中，由 Container Application 在執行時動態組裝**。

**Container Application 的職責**：
- 渲染通用元素（Header、Footer）
- 處理通用功能（Authentication、Shared Libraries）
- 告知每個 Micro-frontend 何時、在何處渲染到頁面上

---

### 兩個常見的混淆

> **混淆一：Micro-frontends 不是 Web Framework**
>
> Micro-frontends 是一種**架構模式**，而非特定的 Web Framework。許多 Framework（如 React、Vue、Angular）都可以用來實作這個模式，但沒有任何一個 Framework 是必需的。

> **混淆二：Micro-frontends 不是 Shared Web Components**
>
> Micro-frontend 不是一個「通用的按鈕」或「搜尋框」——那些是 Shared Components，屬於不同的概念。Micro-frontend 是一個**具有非常具體業務功能的單頁 Web 應用程式**。

---

### 實例：線上學習平台中的 Micro-frontends

以學習平台的「首頁」為例：

1. 使用者發送請求，Web Application Service 將 Container Application 發送給瀏覽器。
2. Container Application 處理 Authentication（並將 Token 存在使用者裝置上）。
3. Container Application 渲染 Header 與 Footer。
4. Container Application 呼叫 Course Discovery Micro-frontend 與 Course Recommendations Micro-frontend 的 Entry Point。
5. 每個 Micro-frontend 向其對應的後端服務發送請求，取得資料、生成 HTML 並渲染到頁面上。

當使用者導航到 User Profile 頁面時：
- Course Discovery 與 Recommendations Micro-frontends **執行 unmount**
- User Profile Micro-frontend（由 User Profile 團隊維護）**執行 mount**，並向 User Service 取得資料

---

### 最佳實踐

#### 最佳實踐一：Runtime 載入，而非 Build-time 依賴

**Micro-frontends 必須在執行時動態載入**，不能是 Container Application 的編譯依賴。

> 否則，我們只是「邏輯上」拆分程式碼庫，執行時仍然是 Monolithic Frontend——任何一個功能的變更仍然需要整個前端重新部署。

這與微服務的道理相同：將 Monolith 應用程式拆分為多個 Modules 或 Libraries，並不能提供與拆分為微服務相同的效益。

#### 最佳實踐二：Micro-frontends 之間不共享 Browser State

若不同 Micro-frontends 需要相互溝通，必須透過以下方式之一：

- **Custom Events**：使用瀏覽器的自訂事件機制
- **Callbacks**：透過 callback 函式傳遞
- **Browser Address Bar**：透過 URL 參數傳遞狀態

> 在瀏覽器中共享狀態，等同於不同微服務之間共享資料庫——從一開始就應該避免。

---

## 💡 重點摘要

- **後端採用微服務但前端仍是 Monolith 時，前端團隊會成為所有後端團隊的共同依賴瓶頸，抵消了微服務帶來的自主性優勢。**
- **Micro-frontends 將 Web 應用拆分為多個由獨立團隊擁有、獨立部署的單頁應用程式，在使用者瀏覽器中由 Container Application 執行時組裝。**
- **Micro-frontends 是一種架構模式而非 Framework，與 Shared Web Components 是完全不同的概念。**
- **Micro-frontends 必須在 Runtime 動態載入，而非作為 Container 的編譯依賴——否則只是「表面拆分」，仍然需要整體部署。**
- **不同 Micro-frontends 之間嚴禁共享 Browser State，跨 Micro-frontend 溝通應使用 Custom Events、Callbacks 或 URL 參數。**

---

## 🔑 關鍵字

Micro-frontends, Container Application, Single Page Application, Runtime Loading, Build-time Dependency, Custom Events, Domain-Driven Design
