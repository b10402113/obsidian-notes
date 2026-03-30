# 第二種 Async 模式：資料複製與事件驅動的完美搭配

## 📝 課程概述

本單元介紹第二種、也是這個課程最終會採用的 **Asynchronous Communication 模式**。這種方式的核心精神是：**每個 Service 維護一份「符合自己需求」的資料副本**，然後透過 Event Bus 在背景持續同步這些資料。最終，Service D 能在完全不需要呼叫其他 Service 的情況下，即時回應幾乎所有的請求。

---

## 核心觀念與實作解析

### 這個模式的核心精神：資料複製

相較於第一種 Async 模式（仍然需要向其他 Service 即時請求資料），第二種 Async 模式的思路完全不同：**讓 Service D 直接擁有自己的資料庫，裡面存放「符合 Service D 查詢需求」的資料**。

這樣做的結果是：Service D 收到請求時，完全不需要向外呼叫任何其他 Service，只要查詢自己的本地資料庫就能立即回應。

---

### Service D 的資料庫應該長什麼樣子？

以「顯示某位使用者曾訂購的所有商品（含標題與圖片）」這個需求為例，Service D 的資料庫可以這樣設計：

**Users Collection（只存必要的欄位）：**
- 記錄每個註冊使用者的 `id`
- 記錄該使用者訂購過的所有商品 `productIds`（陣列形式）

**Products Collection（只存必要的欄位）：**
- 記錄每個商品的 `id`、`title`、`image`

> 這裡有一個非常重要的設計原則：**只複製 Service D 真正需要的資料欄位**，而不需要完整複製來自其他 Service 的所有資料。這也是避免無謂儲存成本的关键。

---

### 如何將資料填充到 Service D 的資料庫？—— Event Bus 的關鍵角色

問題來了：Service D 一開始怎麼會知道有新的使用者註冊、有新的商品上架？

答案在於：**當 Service A、Service B、Service C 各自處理請求時，在完成本地資料庫寫入的同時，會同步向 Event Bus 發出一個事件**。

#### 實作流程

**第一步：建立商品**
- 向 Service B 發送建立商品的請求
- Service B 將商品存入自己的資料庫，**同時**發出一個 `ProductCreated` 事件（包含新商品的 id、title、image）
- Event Bus 將此事件路由給 Service D
- Service D 收到事件後，將這筆商品的 id、title、image 存入自己的 Products Collection

**第二步：使用者註冊**
- 向 Service A 發送註冊請求
- Service A 將新使用者存入自己的資料庫，**同時**發出一個 `UserCreated` 事件（包含新使用者的 id）
- Event Bus 將此事件路由給 Service D
- Service D 收到事件後，在自己的 Users Collection 中建立一筆記錄，初始化 `productIds` 為空陣列

**第三步：使用者訂購商品**
- 向 Service C 發送訂購請求
- Service C 將訂單存入自己的資料庫，**同時**發出一個 `OrderCreated` 事件（包含 userId 與 productId）
- Event Bus 將此事件路由給 Service D
- Service D 收到事件後，找到對應的使用者記錄，將 `productIds` 陣列更新，加入新訂購的 productId

經過這三步，Service D 的資料庫就完整地被填充了。現在，當 client 請求「某位使用者的訂單商品」時，Service D **只需要查詢自己的資料庫**，就能即時回應——完全不需要向外呼叫 Service A、B、C。

---

### 這種 Async 模式的 Pros & Cons

#### ✅ 優點一：Service D 對其他 Service **零依賴**

即使 Service A、B、C 全部掛掉，Service D 仍然可以正常回應查詢請求。這是因為 Service D 所需的資料已經存在自己的資料庫中，不再需要向外請求。

#### ✅ 優點二：**極致的查詢效能**

Service D 的所有資料都在本地，沒有網路呼叫的延遲，理論上可以達到毫秒級甚至更快的回應速度。

---

#### ❌ 缺點一：資料重複（Data Duplication）

這是最多人第一時間反對的點。但讓我們實際算一筆帳：

- 以 AWS、Google Cloud 或 Azure 的 MySQL 為例，**每 GB 儲存的月費僅需 $0.015 ~ $0.0175**
- 以 Amazon 上一個實際商品資訊的 API 回應來說，一筆記錄大約 **1,200 bytes**
- **儲存 1 億筆這樣的記錄，月費約為 $14 美元**

結論：**在現代雲端時代，儲存成本已經低到可以忽略不計**。用少許額外的儲存成本換取整個系統的穩定性與效能，是完全值得的投资。

#### ❌ 缺點二：可能產生資料不一致（Stale Data）

由於 Service D 的資料是透過事件非同步填充的，在極少數情況下可能出現短暫的資料延遲（例如：Service C 發出 `OrderCreated` 事件後，Event Bus 尚未將事件傳遞給 Service D 前，剛好有查詢請求進來）。這是此模式需要接受的一個 trade-off，但後續課程會介紹如何處理這個問題。

#### ❌ 缺點三：概念複雜度較高

需要為每個 Service 設計適合其查詢需求的資料結構，並維護事件發送與接收的對應邏輯。這比簡單的同步呼叫需要更多的前期規劃。

---

## 💡 重點摘要

- 第二種 Async 模式的核心精神是：**讓每個 Service 維護符合自己需求的資料副本，而非即時查詢其他 Service**。
- **事件驅動（Event-Driven）** 是這種模式的關鍵：當某個 Service 發生資料變更時，主動發出事件，其他感興趣的 Service 負責接收並更新自己的資料庫。
- Service D 實現了真正的**零依賴**：即使所有上游 Service 掛掉，Service D 仍可正常運作。
- **Storage 的成本在現代雲端環境中已經極低**，資料重複不是反對這種模式的有效理由。
- 這種模式的代價是**可能存在短暫的資料延遲**，但換來的是效能與穩定性的巨大提升——這是值得的 trade-off。

## 關鍵字

[[Data Replication]], [[Event-Driven Architecture]], [[Database per Service]], [[Data Duplication]], [[Zero Dependency]], [[Storage Cost]], [[Stale Data]], [[Trade-off]], [[Service D]], [[Event Bus]], [[ProductCreated]], [[UserCreated]], [[OrderCreated]]
