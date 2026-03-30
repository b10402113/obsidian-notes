# Event Bus 與 Asynchronous Communication：第一種非同步模式

## 📝 課程概述

本單元介紹第一種 Asynchronous Communication（非同步通訊）模式的核心概念與運作機制，並引入 **Event Bus（事件匯流排）** 這個關鍵元件。我們會看到這種方式如何改變 Service 間的溝通邏輯，同時也會分析它與 Synchronous Communication 相比仍存在哪些缺陷。

---

## 核心觀念與實作解析

### 什麼是 Event Bus？

Event Bus 的核心概念是：引入一個所有 Service 都能連接的共用元件，專門用來處理各 Service 發出的通知與事件。

這些 **Event（事件）** 本質上就是一些描述「應用程式內發生了什麼」的物件，每個事件會包含：
- **type（類型）**：描述這是哪種事件（例如 `UserQuery`、`ProductCreated`）
- **data（資料）**：與事件相關的上下文資訊（例如要查詢的使用者 ID）

---

### Event Bus 的運作機制

1. 每個 Service 連接到 Event Bus 之後，**既可以發送事件，也可以接收事件**。
2. 當某個 Service 向 Event Bus 發送事件時，Event Bus 會自動將該事件的**副本**路由（route）給所有對此事件感興趣的 Service。

### ⚠️ Event Bus 的單點故障風險

因為所有 Service 都連接到同一個 Event Bus，所以 Event Bus 本身成為了整個系統的單一故障點。在部署時，通常需要對 Event Bus 投入額外的 effort 來確保它的穩定性與高可用性。

---

### 第一種 Async 模式的工作流程

回到 Service D 的例子，Event Bus 的應用方式如下：

**情境：Service D 收到「顯示某位使用者訂購的所有商品」的請求。**

1. Service D **發出一個事件**（例如：`UserQuery`，攜帶目標使用者的 ID）
2. Event Bus 將此事件路由給所有對它感興趣的 Service——在這個案例中，會發一份副本給 Service A
3. Service A 接收到事件後，執行查詢（檢查該使用者是否存在），然後**發出另一個事件**（例如：`UserQueryResult`，攜帶使用者資料）回 Event Bus
4. Event Bus 將 `UserQueryResult` 事件路由回 Service D
5. Service D 收到結果後，以同樣的模式向 Service C、Service B 發出事件，取得訂單資料與商品資訊

整個過程中，**Service D 從未直接呼叫任何一個 Service**，而是透過事件的發送與接收來完成資料交換。

---

### 為什麼這種 Async 模式並非首選？

這種方式雖然避免了直接呼叫，但它仍然**繼承了 Synchronous Communication 的所有缺點**，甚至還多了新的問題：

#### 1. Service 間仍有依賴

如果某個事件在處理過程中失敗（例如 Service A 沒有正確處理 `UserQuery` 事件），整個查詢流程同樣會失敗或超時。**Service D 的可用性仍然取決於其他 Service 的健康狀態。**

#### 2. 速度瓶頸不變

整個查詢鏈的響應速度，仍然受限於處理最慢的那個事件環節。

#### 3. 概念複雜度大幅提升

同步呼叫至少還能用一條箭頭直觀表示，而事件驅動的架構需要管理事件的發送、監聽、路由、回應等多個環節，無形中增加了系統的理解與維護成本。

#### 4. 容易形成複雜的事件依賴網絡

當系統中有多個 Service 都在發送和監聽事件時，整個系統的行為變得難以預測和追蹤，特別是當某個 Service 需要處理多種不同類型的事件時。

---

### 結論：第一種 Async 模式是一個起點，但還不是最佳解

這種模式在某些場景下是可用的，但它的缺點足以讓我們繼續探索第二種更進階的 Async 模式——那也是這個課程最終會採用、並在後續專案中大量實作的方案。

---

## 💡 重點摘要

- Event Bus 是所有 Service 共享的通訊中枢，負責事件的路由與分发。
- 事件（Event）由 **type + data** 組成，type 描述事件的種類，data 攜帶相關資訊。
- 第一種 Async 模式仍然沒有消除 Service 間的依賴關係——任何一個環節失敗，會導致整個查詢流程失敗。
- Event Bus 本身是單一故障點，必須額外做好高可用性設計。
- 這種模式的複雜度比同步呼叫更高，但概念上卻不如第二種 Async 模式那樣直觀。

## 關鍵字

Event Bus, Asynchronous Communication, Event-Driven Architecture, Event Type, Event Routing, Service Decoupling, Single Point of Failure
