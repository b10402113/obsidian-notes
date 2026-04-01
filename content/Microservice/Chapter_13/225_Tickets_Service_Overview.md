# Tickets Service 服務概覽

## 📝 課程概述

本章節開始正式構建 Tickets Service，這是我們電子票務系統的核心微服務之一。我們會實作四條 CRUD 路由（Create、Read、Update，以及取得所有票券的 Index），並為這個 service 配置獨立的 MongoDB。本節的重點在於理解整體服務藍圖，以及如何在既有的 auth service 基礎上快速複製出新的 service。

## 核心觀念與實作解析

### 四條路由的設計

Tickets Service 將實作以下四條路由：

| 方法 | 路徑 | 說明 |
|------|------|------|
| `GET` | `/api/tickets` | 取得所有票券（Index） |
| `GET` | `/api/tickets/:id` | 取得特定票券（Show） |
| `POST` | `/api/tickets` | 建立新票券（Create） |
| `PUT` | `/api/tickets/:id` | 更新指定票券（Update） |

**注意：這裡沒有 Delete 路由。** 刪除票券的功能會在後續章節中透過另一個獨立的服務（Orders Service）來處理，而非直接刪除記錄。

### 為何 Price 是 String 而非 Number

建立與更新票券時，`price` 的型別是 **string**，而不是 number。這並不是一個錯誤，而是有特殊原因：票價在真實系統中常涉及小數點與貨幣格式，直接傳送字串可以避免 JavaScript 浮點數精度問題（例：`0.1 + 0.2 !== 0.3`），也便於後續與第三方金流服務整合時直接使用。

### 獨立 MongoDB 的意義

每個微服務應該拥有自己独立的 MongoDB 實例，而不是共用一個資料庫。這是微服務架構的核心原則之一：**每個服務管理自己的資料，服務之間不能直接存取彼此的資料庫**。Tickets Service 的 MongoDB 將存放：

- `title`：活動名稱
- `price`：票價
- `userId`：擁有者的使用者 ID

未來還會加入其他欄位（例如庫存狀態），但這些欄位要等到其他服務建立後才有意義。

### 快速複製 Service 的策略

從頭建立一個完整的 Node.js + TypeScript + Express + Mongoose 微服務大約需要一小時。由於我們已經有 auth service 的完整範本，課程選擇直接複製它的結構：

- `package.json`（修改名稱為 `tickets`）
- `Dockerfile` 與 `.dockerignore`
- `config/` 資料夾
- `src/` 中的 `index.ts`、`app.ts` 及 `test/` 設定檔

複製後，只需要更新少量名稱引用，就能快速啟動一個新的 service，大幅節省時間。

## 💡 重點摘要

- 每個微服務拥有自己獨立的 MongoDB，服務之間嚴禁直接跨庫存取資料。
- `price` 欄位使用 string 是刻意為之，方便處理貨幣精度與外部金流整合。
- 複製現有 service 的程式碼骨架是建立新服務的捷徑，後續再客製化業務邏輯。
- 本服務只實作 Create、Read（單筆與列表）、Update 四個操作，刪除功能由其他服務負責。

## 🔑 關鍵字

MongoDB, Mongoose, Docker, Kubernetes, CRUD, Microservices
