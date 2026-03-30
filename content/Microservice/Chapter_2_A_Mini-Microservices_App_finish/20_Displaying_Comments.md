# N+1 Request Problem：同步通訊解法的缺陷分析

## 📝 課程概述

本單元正式揭露了第一個 microservices 架構下的效能問題——**N+1 Request Problem（N+1 請求問題）**。我們分析了兩種解決方案：同步通訊（Sync）與非同步通訊（Async），並深入探討為什麼同步通訊看似直覺，卻會帶來依賴鏈、效能瓶頸與系統脆弱性等問題。

---

## 核心觀念與實作解析

### 問題的具體表現

當 React App 渲染含有 3 篇文章的頁面時：
1. 對 Post Service 發送 1 次 GET 請求（取得文章列表）
2. 對 Comment Service 發送 3 次 GET 請求（每篇文章各自取得留言）

在這個案例中，N = 3，總共發出了 1 + 3 = 4 次請求。如果有 100 篇文章，就會有 101 次請求。

---

### 解法一：Synchronous Communication（同步通訊）

修改 Post Service，讓它在回傳文章列表時，自動向 Comment Service 發起請求，把留言資料附加到文章物件上再回傳：

```
React App → GET /posts → Post Service
                              ↓ (同步呼叫)
                         Comment Service
                              ↓
                         回傳 comments
                              ↓
                         Post Service 組裝 → 回傳 combined data → React App
```

**優點**：
- 對 client 來說，只需要發送一次請求，概念上很直覺

**缺點（為什麼不選這個方案）**：

1. **引入 Service 之間的依賴**：Post Service 現在依賴 Comment Service。如果 Comment Service 掛掉，Post Service 也跟著無法回應，甚至可能直接 crash。

2. **子請求失敗導致整體失敗**：只要 Comment Service 的回應延遲或失敗，整個請求就會失敗。Client 最終既看不到文章，也看不到留言。

3. **整體速度受限於最慢的環節**：Post Service 的回應時間 = max(Post Service 處理時間, Comment Service 處理時間)。

4. **依賴鏈可能指數成長**：如果 Comment Service 內部還呼叫了其他 Service，那麼一次 GET `/posts` 可能觸發數十甚至數百次隱藏的深層呼叫。只要其中任何一個環節失敗，整個請求失敗。

---

### 為什麼 N+1 問題在 Monolith 中不存在？

在 Monolithic 架構下，所有資料都在同一個資料庫中。要取得「所有文章及其留言」，只需要一條 SQL JOIN 查詢：

```sql
SELECT * FROM posts LEFT JOIN comments ON posts.id = comments.post_id;
```

一次資料庫查詢就能完成所有資料組合。但在 microservices 世界裡，Post Service 和 Comment Service 各自有自己的資料庫，無法跨資料庫直接 JOIN——這正是這個問題的根本原因。

---

## 💡 重點摘要

- N+1 Request Problem 來自於：**每個 Resource 的資料都在各自獨立的 Service / Database 中**。
- 同步通訊看似直覺，但會將多個 Service 串成一條依賴鏈，破壞了 microservices 追求的獨立性。
- 在 Monolith 中，JOIN 一個 SQL query 就能解決的問題，在 microservices 中變成了架構層級的挑戰。
- 同步通訊的代價是：**犧牲 Fault Tolerance（容錯能力）換取實作上的直覺性**。

## 關鍵字

N+1 Request Problem, Synchronous Communication, Service Dependency, Fault Propagation, Bottleneck, JOIN Query, Monolithic vs. Microservices, Cross-Service Request
