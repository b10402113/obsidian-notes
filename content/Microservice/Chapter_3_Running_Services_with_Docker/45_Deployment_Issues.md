# 部署難題：直接擴展架構的三大困境

## 📝 課程概述

本單元從一個最直覺的部署方式切入，展示為什麼把本機開發架構原封不動地搬上虛擬機（VM）會在擴展時遇到瓶頸。我們會看到服務實例增加、跨機器通訊、以及動態擴縮（scale up/down）三個現實場景，如何讓原本看似簡單的 Event Bus 程式碼急速膨脹、最終變得無法維護。這段分析的核心目的，是讓我們在真正介紹 Docker 與 Kubernetes 之前，先理解「為什麼需要它們」。

## 核心觀念與實作解析

### 直接部署到虛擬機：看似可行，實則埋雷

我們先想像最直接的做法：向 Digital Ocean、AWS 或 Azure 租用一台 VM，把所有服務的 source code 複製過去，然後用和本機完全相同的方式啟動（`npm start`），讓各服務透過 `localhost:4000`、`localhost:4001` 等直接溝通。

**這在技術上完全可以運作**——只要你的服務數量永遠不變、流量永遠不變的話。

問題從我們開始認真思考「擴展」的那一刻開始浮現。

### 困境一：同一 VM 內多實例 —— 硬編譯 Port 的代價

假設 comment service 流量暴增，我們決定在同一台 VM 上啟動第二、第三個 instance。此時每個 instance 需要佔用不同的 Port（假設是 `4006` 和 `4007`），而 **Event Bus 必須知道這些 IP + Port**，才能把事件轉發過去。

這就意味著我們必須打開 Event Bus 的程式碼，新增類似這樣的程式碼：

```javascript
// event-bus/index.js（錯誤的做法）
axios.post('http://localhost:4006/events', event); // 手動寫死
axios.post('http://localhost:4007/events', event);
```

> **這樣做的問題**：服務實例數量和 Event Bus 的實作邏輯直接耦合。一旦增減 instance，就必須改 code、重新部署。

### 困境二：跨虛擬機部署 —— IP 管理的地獄

如果單台 VM 的資源也扛不住了，我們租用第二台 VM，把額外的 comment service instance 移過去。這時麻煩來了：

- Event Bus 現在必須同時維護「本機 Port」與「另一台 VM 的 IP + Port」
- 原本的 `localhost` 改成了 `http://<另一台VM的IP>:4006`

```javascript
// event-bus/index.js（更加錯誤的做法）
axios.post('http://<另一台VM的IP>:4006/events', event);
```

> **核心問題**：VM 的 IP 位址並非固定。雲端 VM 重啟後 IP 會改變，每一次變動都要同步更新 Event Bus。這簡直是維護惡夢。

### 困境三：動態擴縮 —— 時間判斷邏輯不該屬於 Event Bus

最後一個場景：網站流量有明顯的波峰波谷（早上 10 點高峰、凌晨 1 點低峰）。我們希望凌晨自動關閉第二台 VM 來省錢。

此時 Event Bus 需要知道「第二台 VM 目前是否活著」——最糟糕的實作方式大概是這樣：

```javascript
// event-bus/index.js（千萬不要這樣做）
const hour = new Date().getHours();
if (hour !== 1) {
  axios.post('http://<另一台VM的IP>:4006/events', event);
}
```

> 這段 logic 汙染了 Event Bus 的核心職責——它應該只管「事件轉發」，而不是「什麼時候轉發給誰」。時間判斷、業務規則和網路層攪在一起，程式碼會快速失控。

### 小結：為什麼我們需要更好的方案

三個困境的共同根源在於：**Event Bus 被迫去「知道」太多細節**——每個服務的位址、它們是否活著、流量何時增減。這是一種高度耦合的設計，終將無法擴展。

我們真正需要的，是一個系統具備以下能力：

- **自動追蹤**所有運行中的服務
- **動態創建或銷毀**服務的副本（instance）
- **自動偵測**服務的存活狀態，自動決定路由

這正是 **Docker** 與 **Kubernetes** 要帶我們脫離的深淵。

## 💡 重點摘要

- 將本地架構直接搬上單一 VM，在服務數量固定時可行，但完全無法應對擴展需求。
- 手動維護 Event Bus 中的 IP + Port 列表，會讓程式碼與部署拓樸高度耦合。
- 跨 VM 通訊若靠硬編譯 IP，雲端 VM 的 IP 變動會導致維護地獄。
- 把流量判斷邏輯寫進 Event Bus，是職責錯亂（Separation of Concerns violation）的經典反例。
- 我們需要一個系統來自動化服務發現（service discovery）、健康檢查（health checking）與動態路由。

## 關鍵字

Deployment Issues, Virtual Machine, Event Bus, Service Discovery, Hardcoded Ports, Load Balancing, Scaling, Dynamic Scaling, IP Address Management, Separation of Concerns
