# Microservices 資料管理難題：Database per Service 模式

## 📝 課程概述

本單元深入探討 Microservices 架構中最大、也最關鍵的挑戰——**資料管理**。我們會認識「一個 Service 一個 Database」的設計模式（Database per Service），並透過 E-Commerce 案例，具體呈現為什麼這條規則看似奇怪，卻是所有成功團隊都在使用的最佳實踐。

---

## 核心觀念與實作解析

### 兩條顛覆直覺的資料規則

Microservices 世界裡有兩條看起來很不直覺的規則：

1. **每一個 Service 在需要時，都會有自己的獨立 Database。**
2. **Service 絕對不能直接讀取另一個 Service 的 Database。**

這兩條規則看起來很不自然，但背後有非常充分的理由。

---

### 為什麼需要 Database per Service？

這個模式背後有三個核心原因：

#### 1. 確保 Service 的獨立性與高可用性

讓我們想像一個反面案例：如果所有 Service 都共享同一個資料庫，那麼一旦這個資料庫出現任何問題（例如：網路中斷、硬碟損壞），**所有 Service 會同時崩潰**。而且，當某個 Service 的負載突然暴增時，我們被迫要去擴展那個共享的資料庫——但事實上可能只有一個 Service 需要更多的資源。

相反的，如果每個 Service 有自己的資料庫，我們可以**只擴展真正需要更多容量的資料庫**，其他服務完全不受影響。

#### 2. 避免 Schema 耦合問題

假設 Service A 的程式碼依賴 Service B 的資料庫結構，Service B 的團隊某天決定修改 schema——例如把 `name` 欄位改名為 `first_name`。如果 Service A 事先不知情，它下次查詢就會拿到預期外的資料格式，進而引發難以追蹤的 bug。

**Microservices 的精神是，每個團隊只管好自己的邊界，Service B 的 schema 變動不該影響 Service A 的運作。** 因此，嚴禁跨 Service 直接讀取資料庫，是保護各服務獨立性的必要手段。

#### 3. 不同 Service 可以選用最適合它的資料庫類型

有些 Service 適合用 MongoDB 的文件模型，有些適合用 PostgreSQL 的關聯式結構。Database per Service 讓每個 Service 都能選擇**對它的工作負載最有效率的資料庫技術**，而不被整體架構限制。

---

### E-Commerce 案例：Monolithic vs. Microservices

讓我們用一個簡單的 E-Commerce 應用程式來對比這兩種架構：

**Monolithic 做法：**
- 所有功能（用戶註冊、商品列表、訂單處理）都在同一個 code base
- 所有資料（users、products、orders）都存在同一個資料庫的不同 tables/collections
- 如果要實作「查詢某位使用者曾經購買的所有商品」這個功能，直接 JOIN 這三張表即可，完全沒有問題。

**Microservices 做法：**
- 用戶註冊 → Service A + 自己的 Database（存放 users）
- 商品管理 → Service B + 自己的 Database（存放 products）
- 訂單處理 → Service C + 自己的 Database（存放 orders）

好，現在問題來了：如果我們要新增一個 Service D，功能是「顯示某位使用者曾購買的所有商品」——**在 microservices 的規則下，Service D 不能直接去讀取 Service A、Service B 或 Service C 的資料庫**。那它該怎麼取得這些資料？

這就是 data management between services 之所以困難的核心原因。

---

## 💡 重點摘要

- **Database per Service** 是 microservices 世界的標準模式，所有成功的工程團隊都在使用這套做法。
- 跨 Service 直接讀取資料庫會造成**緊密耦合**，使得任何一個環節的變動都可能導致連鎖反應。
- 將資料庫各自獨立後，我們才能真正做到 Service 的**獨立部署與擴展**，而不會因為一個環節拖垮整個系統。
- Schema 的變更如果發生在共享資料庫的場景，會造成難以預測的跨服務錯誤。
- 資料管理問題是整個課程的核心聚焦點，後續會介紹兩種主要的解決策略。

## 關鍵字

[[Database per Service]], [[Schema Coupling]], [[Service Independence]], [[Fault Isolation]], [[Data Management]], [[E-Commerce Architecture]], [[Monolithic vs. Microservices]]
