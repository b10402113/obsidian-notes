# Service 間的同步通訊

## 📝 課程概述

本單元介紹微服務之間兩種主要通訊策略中的第一種：**同步通訊（Synchronous Communication）**。我們將看到當 Service D 需要跨 Service 取得資料時，如何透過直接的 HTTP 請求向其他 Service 發問並等待回覆，同時分析這種做法在概念上直觀，但卻帶來依賴鏈、故障傳播與效能瓶頸等重大缺點。

---

## 核心觀念與實作解析

### 同步通訊的定義

首先要注意：**微服務中的 Synchronous / Asynchronous 與 JavaScript 中的同步/非同步是完全不同的概念**，千萬不要混淆。

在微服務的語境下，**Synchronous Communication** 指的是一個 Service 透過直接發送請求（不一定是 HTTP，但通常就是 HTTP + JSON）給另一個 Service 並**等待回覆**的通訊方式。

---

### 同步通訊實作範例

回到 Service D 的需求：**根據 User ID 查詢該用戶訂過的所有商品**。

如果使用同步通訊，Service D 的處理流程會是：

1. **向 Service A 發請求**：確認 User ID 是否存在
2. **向 Service C 發請求**：取得該用戶的所有訂單（進而得知 Product IDs）
3. **向 Service B 發請求**：根據 Product IDs 取得商品的名稱與圖片
4. **合併所有資料後回覆客戶**

> 整個過程中，Service D **從未直接讀取任何其他 Service 的資料庫**，所以我們並沒有違反 Database per Service 的原則。只不過是用「請求另一個 Service」來取代「直接讀取資料庫」。

---

### 同步通訊的優點

#### 容易理解

這是同步通訊最大的優點。整個流程用一張圖就能解釋清楚：Service D 向 A、B、C 各發一個請求，等待回覆，組裝結果。**不需要理解 Event Bus、不需要理解資料同步邏輯**，只要知道「誰有什麼資料，問它就對了」。

#### Service D 不需要自己的資料庫

Service D 完全依賴其他 Service 的回傳資料來回答問題，**因此不需要自己維護一個資料庫**。省去了 Provision、維護與費用。

---

### 同步通訊的缺點

#### 缺點一：建立依賴鏈（Dependency Chain）

這是最大的問題。當 Service D 向 Service A、B、C 發請求，它就**隱式地依賴了這三個 Service 的可用性**。只要其中任何一個 Service 當機，Service D 的請求就會失敗。

> 想像一下：如果你的首頁服務（Service D）依賴了十個其他 Service，只要其中一個故障，整個首頁就開天窗了。

#### 缺點二：單一請求失敗導致整體失敗

如果 Service A 的請求超時或回傳錯誤，Service D **根本無法判斷用戶是否存在**，此時最保險的做法就是回傳錯誤——整個請求就此失敗。

#### 缺點三：整體回應速度受限於最慢的那個請求

假設：
- 查詢 Service A（用戶）：10ms
- 查詢 Service C（訂單）：10ms
- 查詢 Service B（商品）：20,000ms（因為某些原因超慢）

那麼**完成整個請求需要 20,020ms**。服務的回應速度等於最慢環節的速度，這就是所謂的 "Chain of Slow Services" 問題。

#### 缺點四：依賴網絡（Web of Dependencies）失控

這是最容易被忽略但殺傷力最大的問題。在微服務架構中，每個 Service 的內部實作對外是黑箱。當你在開發 Service D 時，你只知道需要問 A、B、C——但你**不知道它們內部又呼叫了誰**。

現實中很可能發生這樣的連鎖：
- Service A 內部呼叫 Service Q
- Service Q 內部又呼叫 Service Z
- Service Z 內部呼叫 Service W

於是，一個來自 Service D 的簡單請求，實際上在後端觸發了**數十甚至數百個隱藏的服務呼叫**。任何一個環節失敗，整條鏈就斷掉。

---

## 💡 重點摘要

- **同步通訊（Synchronous Communication）指的是 Service 透過直接發送請求給其他 Service 並等待回覆，概念直觀，容易理解。**
- **優點是容易實現，且 Service D 不需要自己的資料庫；但缺點是建立依賴鏈、單一失敗會傳播、效能受限於最慢環節，以及隱藏的依賴網絡可能失控。**
- **微服務術語中的 Sync/Async 與 JavaScript 中的同步/非同步是完全不同的概念，不可混淆。**
- **同步通訊適合快速原型或極為簡單的跨 Service 查詢，但不適合對可用性與效能有高要求的生產環境。**

---

## 🔑 關鍵字

[[Synchronous Communication]], [[Dependency Chain]], [[Database per Service]], [[Event Bus]], [[HTTP Request]]
