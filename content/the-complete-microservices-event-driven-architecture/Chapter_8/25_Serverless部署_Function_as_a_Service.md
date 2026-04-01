# Serverless 部署：Function as a Service

## 📝 課程概述

本單元介紹 **Function as a Service（FaaS）**——一種比傳統 VM 部署更徹底的 Event-Driven 部署模式。FaaS 將「Infrastructure 管理」本身也外包給雲端供應商，開發者只需提供「觸發條件」與「執行邏輯」，其余一切——包含擴展與部署——全部由雲端自動處理。這種模式對**事件稀少但流量突發**的工作負擔，有著無可比擬的成本優勢。

---

## 核心觀念與實作解析

### 傳統 VM 部署的成本困境

讓我們用兩個真實場景看清楚 VM 部署在某些場景下的浪費：

**場景一：演唱會售票系統**

- 這個 Microservice 只在演唱會開票時（每個月最多一次）接收大量流量
- 開票後的 30 分鐘到 1 小時內，99% 的票會被售出
- 之後該 Microservice 幾乎完全沒有流量
- **但我們仍然要為整台 VM 全年付費**

**場景二：廣告報表生成系統**

- 每月或每季，由管理員或 Cron Job 觸發一次報表生成
- 假設有 1000 個客戶，每個月（或每季）也只有 1000 次請求
- 這極少量的請求根本不值得長期維持一個完整的 VM 與服務
- 更別說還要維護一堆處理 HTTP Request 的 Boilerplate Code

> 這兩個場景的共同問題：**「有事件才需要做事」的 Microservice，不應該為閒置時間付費。**

---

### Function as a Service 的核心概念

FaaS 是一種從軟體層面到基礎設施層面都完全 Event-Driven 的雲端服務模型。

**開發者只需要提供兩樣東西：**
1. 處理的 **Request / Event 類型**
2. 事件被觸發時要執行的 **Logic**

**雲端供應商負責：**
- 收到事件觸發時，才將程式碼打包、部署到實體硬體上執行
- 流量高峰期自動處理水平擴展（Horizontal Scaling）
- **完全不需要手動配置 Load Balancer 或 Auto-scaling Policy**

> 沒有事件抵達時：**完全不收費**。

---

### FaaS 的定價模型

FaaS 的定價由兩個維度決定：

- **Request 數量**：處理的請求/事件次數
- **執行資源**：每次執行所耗費的時間與記憶體

> 關鍵特性：**只要沒有事件到達，就不收費**。這使得 FaaS 成為「事件稀少但突發」工作負擔的極低成本解決方案。

---

### FaaS 的四大優勢

| 優勢 | 說明 |
|---|---|
| **成本節省** | 對罕見事件、季節性流量突發的工作負擔，費用遠低於長期租用 VM |
| **自動擴展** | 雲端供應商自動處理流量高峰的水平擴展 |
| **降低營運負擔** | 不需要維護 Auto-scaling Policy 與 Load Balancer |
| **降低開發成本** | 雲端供應商負責 Build、Package 與 Deploy，開發者只專注在 Business Logic |

---

### FaaS 的代價：三大限制

#### 限制一：流量模式改變時，成本可能暴增

這是 FaaS 最關鍵的風險：

- 如果一開始是低流量場景，但隨著業務成長，流量越來越大
- Code 執行頻率增加，每次執行因 Business Logic 變複雜而耗費更多時間與記憶體
- **此時 FaaS 的費用可能比長期租用 VM 甚至 Dedicated Host 還要昂貴**

> 正確的使用方式：**選擇 FaaS 前，必須清楚掌握工作負擔的特性。**

#### 限制二：Latency 不可預測

FaaS 的效能穩定性遠低於完全自主控制的 Microservice + VM 部署：

- 流量為零時，FaaS 可能沒有任何運行的 Instance
- 事件到達時，雲端供應商需要「冷啟動（Cold Start）」——打包、部署、啟動一個全新的 Instance
- **冷啟動延遲對延遲敏感的場景是不可接受的**

> 對延遲敏感的場景（如高頻交易、遊戲伺服器、影像串流），FaaS 不是好選擇。

#### 限制三：安全性最低

- **Code 運行在多租戶環境**：與 Multi-tenant VM 的安全風險相同
- **Code 完全暴露給雲端供應商**：需要完全信任供應商的隔離機制

---

### FaaS 與其他部署方式的總結比較

| 維度 | FaaS | Multi-tenant VM | Dedicated Instance/Host |
|---|---|---|---|
| **成本** | 低（罕見事件）→ 高（持續流量）| 穩定可預測 | 最高，但可預測 |
| **彈性擴展** | 自動，完全由雲端控制 | 需配置 Auto-scaling | 需配置 Auto-scaling |
| **Latency 可預測性** | 低 | 高 | 最高 |
| **安全性** | 最低（多租戶 + Code 暴露） | 中 | 高 |
| **適用場景** | 罕見事件、季节性流量突發 | 一般 Web Service、常態流量 | 高合規需求、高效能需求 |

---

## 💡 重點摘要

- **FaaS 的核心價值：「無事件不收費」——對罕見但可能大量的事件（如開票、報表生成），是性價比最高的部署選項。**
- **FaaS 不只是軟體架構的 Event-Driven，更是基礎設施的 Event-Driven——Infrastructure 管理本身也被外包。**
- **FaaS 的最大風險是流量模式改變後，成本可能不降反升——必須持續監控使用量與費率。**
- **Cold Start 問題使 FaaS 不適合延遲敏感的場景。**
- **FaaS 的安全性在三種部署方式中墊底（多租戶 + Code 暴露），敏感資料處理場景應避免使用。**

---

## 🔑 關鍵字

Function as a Service, Serverless, FaaS, Cold Start, Event-Driven Infrastructure, Auto-scaling, Horizontal Scaling, Multi-tenancy, Latency-sensitive
