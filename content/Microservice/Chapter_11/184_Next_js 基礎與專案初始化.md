# Next.js 基礎與專案初始化

## 📝 課程概述

本單元帶你實際動手建立一個 Next.js 專案，包含**安裝依賴**、**撰寫 Script**、以及理解 **Next.js 的檔案系統路由（File-based Routing）** 機制。我們會先在本機端驗證基本功能，稍後再將其部署至 Kubernetes 叢集。

## 核心觀念與實作解析

### 初始化專案

在 `ticketing/` 目錄下新建 `client/` 資料夾，進入後執行：

```bash
npm init -y
npm install react react-dom next
```

Next.js 沒有 Generator，純手工建立所有設定——**這反而讓你能清楚看見所有步驟**，沒有黑魔法。

### package.json 的 Script 設定

```json
"scripts": {
  "dev": "next"
}
```

執行 `npm run dev` 即啟動 Next.js 開發伺服器（預設 port 3000）。

### Next.js 的檔案系統路由

Next.js 的路由**完全由 `pages/` 目錄的檔案結構決定**，不需要像 React Router 那樣另外設定。這是 Next.js 最核心也最優雅的设计之一。

```
pages/
  index.js      → /
  banana.js     → /banana
  auth/
    signup.js   → /auth/signup
```

> `index.js` 代表根路由 `/`；資料夾名稱即為 URL 的路徑區段。

### 為何不使用 TypeScript？

老師解釋：本專案的 React 前端邏輯相對簡單，使用 TypeScript 帶來的型別安全優勢不大，但卻需要耗費大量時間為 Next.js 框架本身添加型別註釋，**得不償失**。所有後端 Service 仍繼續使用 TypeScript。

### Docker 打包設定

Next.js 容器化時需要兩個額外檔案：

**Dockerfile**
```dockerfile
FROM node:alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]
```

**`.dockerignore`**
```
node_modules
.next
```

> **`node_modules` 必須排除**，讓 image 建構時重新 `npm install`；`.next` 是 Next.js 自動產生的快取資料夾，本機端建置產生後不應複製到 image 中。

---

## 💡 重點摘要

- **Next.js 的路由自動對應 `pages/` 目錄的檔案結構，無需額外設定路由表。**
- **`index.js` 對應根路由 `/`；資料夾 + 檔名形成巢狀路徑（如 `/auth/signup`）。**
- **本專案 React 前端不採用 TypeScript，因為框架本身的型別標注成本高於收益。**
- **`node_modules` 與 `.next` 應加入 `.dockerignore`，避免被複製進 image。**

## 🔑 關鍵字

Next.js, File-based Routing, pages/, Dockerfile, node:alpine
