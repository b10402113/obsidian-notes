# 課程自我介紹與 Microservices 入門：什麼是 Microservice？

## 📝 課程概述

本單元是整個课程的開端，我們在這裡建立對 Microservices 的第一個「工作定義」，並透過與傳統 Monolithic Architecture 的對比，快速掌握兩者的核心差異。理解這兩個架構模式的根本不同，是後續所有學習的起點。

---

## 核心觀念與實作解析

### Monolithic Server：一切放在同一個籃子裡

在大多數人接觸過的傳統開發模式中，所有用來實作一個應用程式的程式碼，全都放在同一個 code base 裡面，部署時也是以一個離散的單元（single unit）來進行。

當一個 request 進入 Monolithic Server 時，它的處理流程大致是：

1. Request 先進入某個 preprocessing middleware
2. 抵達 Router，由 Router 根據 request 的內容決定送往哪個 Feature 處理
3. 該 Feature 決定是否要對 Database 進行讀寫
4. 最後組織 response 回傳給 client

如果要用一句話總結 Monolithic 的特性，我們會說：

> **A monolith contains all the routing, all the middleware, all the business logic, and all the database access code required to implement ALL features of our application.**

所有的 routing、middleware、business logic、資料庫存取，全擠在同一個 codebase 裡面。

---

### Microservices：把每個 Feature 包進獨立的 Service

現在，我們把上面那張圖稍微調整一下。Microservices 的定義是這樣的：

> **A single microservice contains all the routing, all the middleware, all the business logic, and database access required to implement ONE feature of our application.**

一個 microservice 只負責實作**一個**功能。每一個 service 都是完全 self-contained（自我包含）的個體——它有自己的 middleware、有自己的 router，**甚至還有自己的 database**。

這裡有個非常重要的觀念要特別強調：**每一個 service 都是 100% 獨立的**。即使其他 service 全部崩潰，某一個 service 仍然可以正常運作。這與 Monolithic 的單點故障（single point of failure）形成了強烈的對比。

---

### 為什麼這個拆分方式很重要？

這麼做的主要目的，是讓每一個 service 可以**獨立運作、獨立部署、獨立擴展（scale）**。當某一個功能的使用量突然暴增時，我們只需要對那個對應的 service 進行擴展，而不需要把整個巨大的 monolith 一起複製一份。

---

## 💡 重點摘要

- Monolithic 把所有功能的程式碼全部集中在一個 codebase，是單點故障的典型代表。
- Microservice 的核心精神是：**一個 service 只實作一個 feature，並擁有自己完整的資源**。
- 每個 service 各自擁有獨立的 database，這是 microservices 架構中非常重要的設計原則。
- Service 之间保持独立，意味着系统整体的 **fault tolerance（容錯能力）** 大幅提升。
- 這個課程的核心目標，就是解決「多個 service 之间如何共享與管理資料」這個難題。

## 關鍵字

[[Monolithic]], [[Microservice]], [[Single Unit Deployment]], [[Self-Contained Service]], [[Database per Service]], [[Fault Tolerance]], [[Feature Isolation]]
