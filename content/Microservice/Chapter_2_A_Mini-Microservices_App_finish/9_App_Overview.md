# 迷你 Microservices 應用程式：需求分析與架構規劃

## 📝 課程概述

本單元揭開本課程第一個實作專案的序幕。我們會建立一個類似 Blog 的簡易應用程式，具備「發表文章」與「新增留言」兩大功能。這個專案的目的不是打造 production 級系統，而是**純手工實作 microservices 的核心溝通機制**，讓你近距離體會非同步事件驅動模式的工作原理與潛在挑戰。

---

## 核心觀念與實作解析

### 為什麼要手工實作，而不用現成套件？

在這個迷你專案裡，我們會自己刻出所有的 Service、自己實作 Event Bus，不依賴任何額外的框架。這樣做的原因是：**我們要從最底層理解這些東西到底在解決什麼問題**。等到後續更完整的專案時，你才會看到業界實際使用的套件該如何選擇與搭配。

> 這個迷你專案千萬不要當成未來其他專案的範本——它是學習用的練習場，不是 production 的範例。

---

### 應用程式的功能需求

使用者的操作流程如下：

1. **建立文章（Post）**：在最上方的表單輸入文章標題（只有標題，沒有內文），點擊 Submit
2. **畫面下方立刻顯示新文章**：包含文章標題，以及該文章的留言計數
3. **為特定文章新增留言**：在每篇文章下方有一個表單可以新增留言，Submit 後：
   - 該文章的留言計數馬上 +1
   - 該文章下方即時列出新的留言內容
4. 可以新增任意數量的文章，並對每篇文章各自留言

---

### Service 的劃分原則：Resource-Based Design

我們的共識是：**每一種不同的 Resource，就建立一個獨立的 Service**。這堂課的應用程式有兩種 Resource：

| Resource | Service | 負責的工作 |
|----------|---------|-----------|
| Post（文章） | Post Service | 建立文章、列出所有文章 |
| Comment（留言） | Comment Service | 建立留言、列出某篇文章的所有留言 |

---

### Comment Service 的隱藏複雜度

Post Service 的職責非常直覺，但 Comment Service 就不那麼單純了，原因在於**留言必須與特定的文章綁定**。這帶來了幾個挑戰：

1. **建立留言時**：要確保留言關聯到的是一個真正存在的文章 ID——這就需要通訊機制。
2. **列出留言時**：我們**不希望**列出「所有文章的所有留言」，而是**只列出某一篇文章的留言**——這同樣需要跨 Service 的資料查詢。

換句話說，Comment Service 需要「知道」某個 Post 是否存在、某個 Post 有哪些留言——這些資訊都在別的 Service 手裡。這就是為什麼我們需要第一堂課所學的通訊策略（Sync vs. Async）。

---

## 💡 重點摘要

- 這個迷你專案是學習工具，不是 production 範本——我們會手工實作所有元件。
- Resource-Based Design 是 microservices 設計中常見的拆分策略：**每種 Resource 一個 Service**。
- Comment Service 的難點在於它天然依賴 Post Service 的資料，但 microservices 的原則要求 Service 不能直接讀取彼此的資料庫。
- 這個應用程式看似簡單，但實作時會暴露出不少設計與通訊上的挑戰。

## 關鍵字

Resource-Based Design, Post Service, Comment Service, Create Post, List Comments, Data Synchronization, Microservices Architecture, Event-Driven
