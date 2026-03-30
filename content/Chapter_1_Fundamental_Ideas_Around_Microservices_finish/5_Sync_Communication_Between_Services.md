# Service 間的溝通策略：Synchronous Communication（同步通訊）

## 📝 課程概述

本單元介紹 microservices 世界中兩種主要的服務間溝通策略——**Synchronous（同步）** 與 **Asynchronous（非同步）** 通訊。我們的重點在於理解這兩種方式各自的運作機制、優缺點，以及在實務上的取捨。這裡要特別提醒：microservices 領域中「sync / async」的定義，**與 JavaScript 世界中的非同步概念完全不同**，千萬不要混淆。

---

## 核心觀念與實作解析

### Synchronous Communication 的定義

在 microservices 的語境下，Synchronous Communication 指的是：**一個 Service 透過直接發送 Request 的方式，向另一個 Service 取得資料**。

這個 Request 不一定是 HTTP Request，也不一定要交換 JSON，它可以是任何形式的直接請求。核心精神只有一個：**一個 service 直接向另一個 service 發起呼叫，要求對方回應資料**。

---

### 運作範例：Service D 向 A、B、C 三個 Service 請求資料

延續前面的 E-Commerce 案例，當 Service D 收到「顯示某位使用者訂購的所有商品」的請求時：

1. Service D 直接向 **Service A** 發送 Request，詢問「這個 user ID 是否存在」
2. Service A 回應後，Service D 再向 **Service C** 發送 Request，查詢該使用者的所有訂單
3. 最後向 **Service B** 發送 Request，取得這些商品的詳細資訊
4. Service D 蒐集完所有回應後，才能完成並回傳最終結果

在這個過程中，**Service D 完全沒有直接碰觸任何其他 Service 的資料庫**，完全符合 database per service 的紀律。

---

### Synchronous Communication 的優點

- **易於理解**：不需要太多額外的抽象概念，只要看圖就能明白資料的流向。
- **Service D 不需要有自己的 Database**：因為它隨時可以向其他 Service 即時查詢，所以不需要另外維護一份資料。

---

### Synchronous Communication 的缺點

這才是這種方式的關鍵問題所在：

#### 1. 引入 Service 間的依賴關係

如果 Service A、B、C 中任何一個掛掉了，Service D 也會跟著無法運作。因為 Service D 的 request 勢必要依賴這些服務的回應——只要其中一個環節失敗，整個請求就失敗。

> **這和我們一開始追求的「每個 Service 獨立運作」的目標，是直接衝突的。**

#### 2. 任何一個子請求失敗，整體請求就跟著失敗

這和第一點的道理相同。若向 Service A 的請求超時或失敗，Service D 別無選擇，只能向 client 回報錯誤。

#### 3. 整體速度取決於最慢的那個 Request

假設：
- 向 Service A 的請求：10ms
- 向 Service C 的請求：10ms
- 向 Service B 的請求：**20 秒**

那麼 Service D 處理這個請求的總時間，就是 **20 秒 + 20ms**。你的系統的回應速度，永遠受限於那個最慢的環節。

#### 4. 依賴鏈可能形成複雜的「Web of Dependencies」

這是最容易被低估的問題。我們可能知道 Service D 需要呼叫 A、B、C，但 A 的內部實作可能又呼叫了 Service Q，Q 又呼叫了 Z 與 X。表面上只有三個呼叫，事實上背後可能隱藏著數十甚至數百個深層的 request。

只要這整條鏈中的**任何一個節點失敗**，整個請求就會失敗。而且速度永遠受限於最慢的那一環。

---

### ⚠️ 重要的觀念澄清：JavaScript 的 async ≠ Microservices 的 Async

在 JavaScript 中，`async/await` 或 Callback 描述的是「程式執行時不阻塞主執行緒」的機制。在 microservices 領域中，Async 的意義完全不同——它指的是「發送端發出請求後，不需要等待接收端立即回應」的通訊模式。兩者千萬不要搞混。

---

## 💡 重點摘要

- Synchronous Communication 的精神是：**直接、立即、双向（request → response）** 的溝通模式。
- 最大的問題是：**引入 Service 間的依賴**，破壞了 microservices 追求的獨立性原則。
- 最慢的那個 request 會拖累整個請求鏈的響應時間，這在高併發場景下是致命的。
- 隱藏在依賴鏈後面的深層呼叫，往往是系統不穩定的隱患。
- Microservices 中的 sync/async 與 JavaScript 的同步/非同步是完全不同的概念。

## 關鍵字

[[Synchronous Communication]], [[Request-Response Pattern]], [[Service Dependency]], [[Web of Dependencies]], [[Performance Bottleneck]], [[Fault Propagation]], [[Database per Service]]
