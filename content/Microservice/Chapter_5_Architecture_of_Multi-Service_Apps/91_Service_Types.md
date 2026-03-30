# 服務類型與系統架構設計

## 📝 課程概述

本單元正式進入第二個應用程式的架構規劃階段。我們根據上一堂課定義的四種資料模型（User、Ticket、Order、Charge），設計出五個各有專職的 microservice，並引入本課程的幾項重大技術升級：MongoDB 作為持久化儲存、Redis 處理限時鎖定、NATS Streaming Server 作為企業級 Event Bus、以及一個全課程通用的共享 library（common）。在正式動手之前理解這套架構的全貌，是後續開發的地圖。

## 核心觀念與實作解析

### 五個核心服務的職責分配

#### Auth Service（認證服務）

專門處理所有與使用者身份驗證相關的流程：註冊（Sign Up）、登入（Sign In）、登出（Sign Out）等。這是整個應用程式的守門員，負責建立和管理 User 資源。

#### Tickets Service（票券服務）

負責 Ticket 的**建立與編輯**。它完整掌握每張票券的狀態——價格是多少、目前是否被鎖定、誰是擁有者等。任何建立票券或修改票券內容的行為，都由這個服務處理。

#### Orders Service（訂單服務）

負責 Order 的**建立、修改與管理**。當使用者點擊 Purchase 時，這個服務會建立一個 Order 物件並設定 `expiresAt` 過期時間，同時負責更新 Order 的狀態（AwaitingPayment / Cancelled / Complete）。

#### Expiration Service（過期服務）

這是一個**專門監聽事件**的服務——監聽任何新 Order 被建立的時刻，並在 15 分鐘後自動執行取消訂單的邏輯。這個服務的設計非常精簡，但職責明確：計時器到期就觸發取消，不多不少。

#### Payments Service（付款服務）

處理信用卡的**實際扣款**。當使用者輸入信用卡資訊並成功請款時，通知 Order 改為 Complete；若 Order 被取消或逾時，則同步將付款標記為 Failed 或執行退款。

---

### 服務拆分策略：「一個資源一個服務」的取與捨

講師特別強調，這個應用程式採用「一個 Service 管理一種 Resource」的設計，**並不意味這是所有 microservice 架構的唯一正確做法**。

#### 為什麼我們這樣設計？

在課程的語境下，這是最容易理解、最直觀的拆分方式。四種資料模型、對應四（或五）個服務，每個服務的職責邊界清晰，學員可以專注在 microservice 的核心觀念，而不會被過度複雜的服務邊界問題困擾。

#### 現實世界的替代做法

在實際產品開發中，你可能會根據**業務邏輯的耦合程度**來決定服務邊界。例如：
- 一個服務同時處理 Ticket 和 Order 的建立（因為兩者高度相依）
- 一個服務專門處理 Payment 的整個生命週期（包含 Order 的狀態更新）
- Expiration 邏輯可能直接內嵌在處理 Order 的同一個服務中

> **核心原則**：永遠根據**你的應用程式需求**來設計服務邊界，而不是盲目套用「一個資源一個服務」的金科玉律。把「一個 Service 管理一種 Resource」當作起點，再根據業務邏輯的耦合程度去做調整。

---

### 系統架構圖解析

```
[Next.js Client]
       ↓
┌─────────────────────────────────────────┐
│          NATS Streaming Server          │
│              (Event Bus)                │
└─────────────────────────────────────────┘
       ↑          ↑          ↑          ↑
   [Auth]    [Tickets]   [Orders]  [Payments]
             [Expiration]
                 ↑
            [MongoDB]      [MongoDB] ...  (每個服務各自的 MongoDB)
                 ↑
            [Redis] (僅 Expiration 使用)
```

#### Next.js（React Client）

前端採用 **Next.js** 而非傳統的 React SPA。Next.js 是一個具備 Server-Side Rendering（SSR）能力的 React 框架。在電商與票券系統中，SSR 能讓使用者在首次載入時更快看到內容，且對 SEO 有幫助。

#### 各服務：Node + Express + MongoDB

每個 service 都是一個 Node.js Express 應用程式，各自擁有獨立的 **MongoDB** 資料庫。這與第一個專案在記憶體中儲存資料的做法完全不同——資料會持久化，服務重啟不會造成資料遺失。

#### Redis：用於 Expiration Service 的特殊需求

為什麼 Expiration Service 需要 Redis？講師預告了答案：**Redis 的 keyspace notification** 或 **TTL（Time-To-Live）機制**可以優雅地處理「15 分鐘後自動取消訂單」的計時需求，而不需要自己寫一個 `setTimeout` 或定時輪詢。這是一個非常適合 Redis 的使用場景。

#### Common Library（共享 Library）

這是我們在痛點回顧中提到的解決方案之一。建立一個名為 `common` 的 **NPM 模組**，裡面放置：
- 共享的 Express middleware（如驗證 middleware）
- 所有 Event 的類型定義與結構
- 共用的工具函式

所有五個服務都會 `npm install common`，確保事件名稱、屬性型別等資訊在所有服務之間**高度一致**。

#### NATS Streaming Server：企業級 Event Bus

不再使用第一個專案中自己手工打造的 Event Bus。這次選用 **NATS Streaming Server** 作為事件匯流排，負責：
- 接收來自各服務的事件
- 將事件廣播給所有有興趣的服務

> 注意：NAT Streaming Server 與網路領域的 NAT（Network Address Translation）完全是兩回事，不要混淆。

---

### 即將處理的事件清單預覽

講師預先展示了系統會用到的事件清單（部分）：
- `UserCreated`、`UserUpdated`
- `TicketCreated`、`TicketUpdated`、`TicketDeleted`
- `OrderCreated`、`OrderCancelled`、`OrderExpired`、`OrderCompleted`
- `PaymentCreated`、`PaymentFailed`

這些事件的名稱大都可以「望文生義」，但每個事件的**完整定義與觸發時機**，會隨著課程進展逐步深入探討。現在只需要知道：有這些事件存在，且它們都會在 `common` library 中被嚴格定義。

---

## 💡 重點摘要

- 「一個 Service 管理一種 Resource」的設計是**出於教學考量**的選擇，而非 microservice 的絕對原則，現實中應根據業務耦合度做調整。
- Expiration Service 選用 Redis，是因為 Redis 的 TTL/Keyspace Notification 非常適合用來實作「計時後自動觸發」的邏輯。
- NATS Streaming Server 是一個功能完整的企業級事件匯流排，比自己實作的版本更穩健且易於擴展。
- `common` library 是確保整個系統事件定義一致性的核心，必須在所有服務開始實作**之前**就建立完成。
- 這個應用程式的每個 service 都會有自己獨立的 MongoDB，資料不再存在記憶體中。

## 關鍵字

Auth Service、Tickets Service、Orders Service、Expiration Service、Payments Service、MongoDB、Redis、NATS Streaming Server、Next.js、Common Library、SSR、TTL、Event Bus、Service Boundary Design、Shared Library、microservice Architecture
