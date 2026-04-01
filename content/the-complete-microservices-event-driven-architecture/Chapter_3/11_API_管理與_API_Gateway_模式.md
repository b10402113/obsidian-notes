# API 管理與 API Gateway 模式

## 📝 課程概述

當系統中存在數十甚至數百個微服務，各自暴露不同的 API 時，API 管理會迅速失控。本單元介紹 **API Gateway Pattern（API 閘道模式）**——在系統入口處放置一個統一元件，負責所有 API 管理工作（路由、轉換、限流、授權、監控），讓微服務從這些繁瑣的横切關注點（Cross-Cutting Concerns）中解放出來，專注於自身的商業邏輯。

## 核心觀念與實作解析

### API 管理在微服務架構中的挑戰

讓我們以一個醫療健康平台為例，該公司為健身中心、醫院、診所等提供數位服務。系統中有：

- **多種客戶端**：智慧手錶、手機、網頁瀏覽器、IoT 裝置、合作夥伴伺服器等
- **數十到數百個微服務**，各自暴露針對其功能的專屬 API

在這種規模下，API 管理會面臨多個具體問題：

#### 問題一：客戶端程式碼與內部實作緊密耦合

不同微服務暴露的不同 API endpoint 被**直接寫死在各類前端與 Client SDK 中**。例如，當我們決定將某個微服務拆分為兩個獨立的微服務時，所有使用該 API 的客戶端程式碼都必須跟著重構——這是一種極為脆弱的耦合。

#### 問題二：同一服務需要多種 API 版本

同一個微服務可能需要同時支援：
- **公開 API**：提供給外部客戶
- **私人 API**：僅提供給內部服務
- **合作夥伴 API**：僅提供給合作企業與開發者

更複雜的是，同一個合作夥伴可能只支援舊版 API 技術，而另一個合作夥伴支援新版技術；我們可能還需要針對不同訂閱層級的客戶提供不同能力的 API（如免費版 vs. 付費版的差異化 API）。

#### 問題三：跨多個微服務的追蹤與監控困難

一個使用者打開健身數據 Dashboard，行動 App 可能需要呼叫多個不同的微服務。但因為前端直接呼叫各服務，要對整個呼叫鏈路進行追蹤與監控極為困難。

#### 問題四：Boilerplate 程式碼的重複

Authorization、Rate Limiting、TLS 終止（Termination）等横切關注點必須在**每個微服務中各自實作**——造成大量重複，也浪費了每個團隊本可專注於商業邏輯的時間。

---

### API Gateway Pattern：統一入口的解決方案

#### 什麼是 API Gateway？

API Gateway 是在系統入口處放置的一個元件，**負責所有 API 管理職責**。在最簡單的情境下，它只做請求路由——將收到的 API 請求轉發到對應的微服務。但實際上它遠不止於此。

#### API Gateway 的核心能力

| 能力 | 說明 |
|------|------|
| **Protocol & Data Translation** | 將來自不同客戶端的請求統一轉換為內部服務支援的標準格式，並在回應時再轉換回去 |
| **Traffic Management & Throttling** | 在 Gateway 層對高流量客戶端進行限流，防止其對後端服務造成過載 |
| **Authorization & TLS Termination** | 由 Gateway 統一處理，服務專注於商業邏輯 |
| **Monitoring & Analytics** | 所有流量都經過 Gateway，可以輕鬆偵測特定 API 的問題或分析客戶端行為 |
| **Request Fan-out & Aggregation** | 將一個請求同時發送給多個服務，彙整結果後再回傳給客戶端 |

#### Fan-out 能力的重要性

Request Aggregation（請求彙整）是 API Gateway 的一個關鍵能力，原因有兩個：

1. **客戶端與內部實作完全解耦**：我們可以在內部自由地將多個微服務合併為一個，或將一個微服務拆分為多個——這些實作細節對客戶端完全透明。
2. **對 Mobile 與 IoT 設備極為重要**：這些設備的每次網路請求都直接影響電池壽命。Gateway 的彙整能力減少了客戶端需要發起的請求數量。

---

### API Gateway vs. Load Balancer：釐清混淆

這是課堂中最容易混淆的話題，老師特別做了明確的比較：

| 維度 | Load Balancer | API Gateway |
|------|--------------|-------------|
| **目的** | 將流量均衡分發到一組伺服器（通常是相同應用程式的多個實例） | 作為系統的公開面向介面，根據不同條件將請求路由到**服務**（而非伺服器） |
| **部署位置** | 通常放在每個微服務前方，每個微服務部署為一組相同實例 | 放在整個系統的最前端，所有外部流量統一入口 |
| **設計哲學** | 盡量簡單以減少效能開銷，通常附帶 Health Check 與多種負載均衡演算法 | 複雜得多，提供 Throttling、Monitoring、API Versioning、Protocol Translation 等眾多功能 |
| **微服務中的典型使用** | 與 API Gateway 共同使用 | 與 Load Balancer 共同使用 |

> **關鍵理解**：兩者的目標讀者不同——Load Balancer 的目標讀者是**基礎設施團隊**（管理伺服器），API Gateway 的目標讀者是**應用程式開發者**（管理服務與 API）。它們服務於不同的目的，因此在微服務架構中**兩者通常同時存在**。

---

## 💡 重點摘要

- **當微服務數量龐大時，直接讓客戶端呼叫各服務 API 會造成緊密耦合、版本混亂與監控死角——這就是 API 管理問題的根源。**
- **API Gateway 在系統入口處統一處理 Protocol Translation、Throttling、Authorization、Monitoring 等横切關注點，讓微服務專注於商業邏輯。**
- **Request Aggregation（請求彙整）是 API Gateway 的殺手級功能：它讓內部實作演進對客戶端完全透明，並減少 Mobile/IoT 設備的請求次數以節省電量。**
- **Load Balancer 服務於基礎設施層（均衡流量到伺服器實例），API Gateway 服務於應用層（管理服務間的 API 路由與治理）——兩者分工不同，通常一起使用。**
- **沒有 API Gateway，每個微服務都必須實作 Authorization、Rate Limiting 與 TLS Termination——這些 boilerplate 不僅浪費開發時間，更增加了系統的安全風險與不一致性。**

---

## 🔑 關鍵字

API Gateway, Load Balancer, Throttling, TLS Termination, Request Aggregation, Rate Limiting, Monitoring, API Versioning
