# Contract Tests 與生產測試

## 📝 課程概述

本單元針對上一堂課提出的三大挑戰（E2E Tests 成本過高、Integration Tests 造成團隊耦合、Event-Driven 架構測試困難），介紹具體的替代方案：Contract Tests（契約測試）用來取代複雜的 Integration Tests，以及基於 Blue-Green Deployment 與 Canary Testing 的**生產測試（Testing in Production）**策略作為 E2E Tests 的替代方案。

## 核心觀念與實作解析

### 簡化 Integration Tests：Lightweight Mocking

先從最簡單的優化說起。假設團隊 A 擁有微服務 A（Consumer），欲對微服務 B（Provider）的 API 執行 Integration Tests。

**傳統做法**：必須啟動微服務 B 及其所有依賴（Database、其他 Service）。
**Lightweight Mocking 做法**：只 Mock 微服務 B 的 API 層——讓 Mock 層在收到預期中的請求時，回傳寫死的標準響應。

同樣地，微服務 B（Provider）的團隊可以用 Mock Consumer 代替真實的 Consumer 微服務來執行測試，驗證「當收到某請求時，是否回傳正確響應」。

**好處**：一個團隊 Build 失敗不會阻斷另一個團隊的測試，且省去了啟動真實微服務實例的開銷。

**但 Lightweight Mocking 本身有一個致命缺陷**：若 API Provider 團隊變更了 API、並更新了 Mock 與自己的測試，雙方測試都會通過——**但這個 API 變更可能沒有被正確溝通給 Consumer 團隊**，雙方在生產環境中會完全無法溝通，造成系統故障。

---

### Contract Tests：自動維持 API 契約同步

Contract Tests 的出現就是為了解決「Mock 與真實實作不同步」的問題。

#### 運作機制

Contract Tests 使用專門的工具，在 API Consumer 與 API Provider 之間維護一個**共享的 Contract File（契約檔案）**：

1. **Consumer 端測試執行時**：除了測試自己的商業邏輯外，還會將「發送給 Mock Provider 的請求」與「預期的響應」一併**記錄到 Contract File** 中。
2. **Provider 端收到 Contract File 後**：Replay（重放）所有記錄的請求到**真實的微服務 B** 上，驗證響應是否與 Contract File 中記錄的一致。
3. 若微服務 B 有多個 Consumer，就對每個 Consumer 的 Contract 分別建立 Mock Consumer，逐一執行驗證。

```
Consumer Team              Contract File              Provider Team
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│ Mock Provider │◄────────│  Shared      │────────►│ Real Service │
│ + Recorder    │         │  Contract    │         │ B            │
└──────────────┘         └──────────────┘         └──────────────┘
```

> **關鍵保障**：Contract Test 工具會確保 Consumer 端測試所基於的 Mock 與 Provider 端測試所驗證的真實實作**始終保持同步**。任何一方對 API 的變更，都會在另一方的測試中被即時發現。

**Contract Tests 的核心價值**：每個團隊可以獨立執行自己的 Integration Tests，**無需建置、設定或執行其他團隊的微服務**——但同時保證了雙方測試所依據的契約是最新的、正確的。

---

### 將 Contract Tests 擴展至 Event-Driven 架構

Contract Tests 的概念同樣可以應用於透過 Message Broker 進行非同步通訊的微服務。

以電子商務系統為例：
- 使用者下單 → Order Service 發布事件到 Message Broker
- Payment Service 消費該事件，處理後再發布事件到 Shipping Service

**Event Contract 的內容**：事件格式（包含使用者、產品、數量等資訊）。

**測試流程**（以 Payment Service 與 Shipping Service 之間的整合為例）：

1. Shipping Service 團隊執行測試時，嘗試解析 Payment Service 發出的事件，同時**將該事件格式與內容記錄到 Contract File**。
2. Payment Service 團隊取得 Contract File 後，觸發自己服務中「應該產生該事件」的函式，由 Contract Test 工具**驗證產生的事件格式與內容是否與 Contract File 中記錄的一致**。
3. 若雙方格式一致，則 Event Contract 處於同步狀態，可以自信地部署到生產環境。

> **為什麼這麼有效？** Contract Tests 將「需要啟動整個 Message Broker + 上下游服務」的超高成本測試，簡化為「本地記錄事件格式 → 交換 Contract File → 各自執行輕量級驗證」。

---

### 取代 End-to-End Tests：Testing in Production

Contract Tests 可以大幅簡化 Integration Tests，**但它們並不能取代 End-to-End Tests**。若公司無法負擔完整的 E2E 測試環境，另一個可行策略是：**直接在生產環境中測試**。

#### Blue-Green Deployment

Blue-Green Deployment 是一種零停機發布新版本微服務的方式，核心思想是**維護兩個完全相同的生產環境**：

- **Blue 環境**：運行舊版微服務的伺服器/容器集合
- **Green 環境**：部署新版本微服務的伺服器/容器集合

發布流程：
1. 將新版本部署到 Green 環境（**尚無真實用戶流量**）
2. 在 Green 環境上執行自動化或手動測試
3. 若測試通過，逐步將部分流量導向 Green 環境（這就是 Canary Testing）
4. 若偵測到問題，立即將流量全部導回 Blue 環境（影響極小）
5. 若一切正常，最終將所有流量導向 Green 並退役 Blue 環境

#### Canary Testing

Canary Testing 是「讓一小部分真實用戶先試用新版本」的策略。這個名稱來自煤礦工人的比喻——帶著金絲雀進礦坑探測毒氣：

- 一開始只將 5% 的流量導向新版本
- 監控新版本的效能與功能是否正常
- 若正常，逐步增加比例至 100%
- 若有異常，瞬間切回舊版本

> **為什麼比 E2E Tests 更有效？** 因為使用的是**真實的生產流量**（而非人為建構的測試案例），更能反映實際使用情境中的邊界條件與錯誤。

---

## 💡 重點摘要

- **Lightweight Mocking 簡化了 Integration Tests 的設定成本，但無法防止 API Provider 與 Consumer 之間的契約悄悄失去同步——這正是 Contract Tests 存在的意義。**
- **Contract Tests 透過共享的 Contract File，讓 Consumer 端與 Provider 端都能針對「同一份契約」各自執行測試，既保留團隊獨立性，又確保雙方實作始終同步。**
- **Contract Tests 的概念可以自然延伸至 Event-Driven 架構：Producer 端記錄事件格式，Consumer 端驗證是否能正確解析——完全不需要啟動 Message Broker。**
- **當完整的 E2E 測試環境成本過高時，Blue-Green Deployment + Canary Testing 提供了一個務實的替代方案：用真實流量代替測試案例，在生產環境中以最小風險驗證新版本。**
- **所有這些替代方案都只是「在無法建立完整測試環境時」的務實選擇——若資源允許，Contract Tests 與 Production Testing 應該與傳統測試金字塔共存，互為補充。**

---

## 🔑 關鍵字

Contract Tests, Pact, Blue-Green Deployment, Canary Testing, Testing in Production, Event Contract, Mocking, E2E Tests
