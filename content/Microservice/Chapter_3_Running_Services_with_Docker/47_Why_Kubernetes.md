# 為什麼需要 Kubernetes：服務叢集管理與通用通訊層

## 📝 課程概述

本單元承接前一章對部署困境的分析，引入 Kubernetes 這個開源工具的核心概念。的重點在於：Kubernetes 如何讓我們從「Event Bus 必須知道所有服務位址」的泥沼中解脫出來，轉而提供一個統一的、通訊雙方都不需要相互知道彼此存在的神奇「中介層」。同時，我們也會對 Docker 的定位做一個快速回顧。

## 核心觀念與實作解析

### Docker 回顧：封裝執行環境

在談 Kubernetes 之前，先快速回顧 Docker 的價值主張。

我們的微服務應用在本地啟動時，有幾個隱含假設：
- **Node.js** 已安裝
- **npm** 已安裝
- 每個服務有不同的啟動命令（`npm start`）

這些假設對開發者本人來說或許不痛不癢，但當你需要把同一套程式碼部署到不同的機器、不同團隊的環境、不同雲端供應商時，這些「隱含假設」就會一個個爆開。

Docker 的解法是：**把一個程式所需的一切（作業系統、依賴套件、啟動命令）全部封裝進一個獨立的 Container**。

- Container 內包含了 Node.js 與 npm
- Container 內也包含了「如何啟動這個程式」的說明

如此一來，**只要能跑 Docker 的地方，就能原封不動地運行我們的服務**。這解決了「環境一致性问题（Environment Consistency）」。

### Kubernetes 是什麼？

> **Kubernetes 是一個用來同時運行、管理多個 Containers 的工具。**

當我們把 Docker Container 想像成「包裝好的程式」之後，Kubernetes 的角色就清晰了：**它負責把這些包裝好的程式在叢集（Cluster）上啟動起來、管理它們之間的網路通訊、以及自動處理各種故障與擴縮需求。**

### Cluster 架構：Node 與 Master

一個 Kubernetes Cluster 由以下角色組成：

- **Node（節點）**：即虛擬機。可以是 1 台、100 台、甚至上千台。Node 負責實際執行 Containers。
- **Master（主節點）**：一個特殊的程式，管理和協調整個 Cluster 內的所有 Node——追蹤哪些 Node 活著、調度新 Containers 要放到哪個 Node、以及維護整個系統的期望狀態。

我們對 Kubernetes 做的事情本質上是：**寫 YAML 設定檔，告訴 Master 我們想要幾份某個服務的副本、它們該怎麼互相發現，Master 就會自動幫我們執行到位。**

### Kubernetes 如何解決 Event Bus 的困境？

想像一下，我們在 Cluster 裡放了：
- 兩份 Post Service（各在一個 Container 內）
- 一個 Event Bus（第三個 Container）

如果沒有 Kubernetes，Event Bus 必須分別知道「第一份 Post Service 在哪個 Node、哪個 Port」以及「第二份在哪裡」。這就是我們前一章看到的泥沼。

**Kubernetes 提供了一個統一的通訊層**。我們可以想像成：Event Bus 不再試圖直接呼叫某個特定的 Post Service instance，而是把所有請求發到一個「中介地址」——由 Kubernetes 自動將請求轉發給目前 Cluster 中所有運行中的 Post Service 副本。

> 這就是所謂的 **Service Discovery** 自動化：**服務不需要知道彼此的具體位址，只需要知道「有這個名字的服務存在」，Kubernetes 會自動處理路由。**

### 為什麼 Kubernetes 特別適合微服務？

1. **動態擴縮（Dynamic Scaling）**：需要更多 Comment Service instance？改一行設定，Kubernetes 自動啟動新的 Containers，並自動更新路由——Event Bus 完全不需要知道有多少實例。
2. **健康檢查（Health Checking）**：當某個 Container 當機，Kubernetes 會自動偵測並重啟它。
3. **跨 Node 網路**：無論 Container 落在哪個 Node，Kubernetes 的網路層都會確保它們可以相互通訊，不需要你手動維護 IP 清單。

### 嚴格說來：服務之間並不是「完全不溝通」

課程在這裡特別澄清一個常見的誤解：

> 我們說「不該用同步（Synchronous）呼叫讓服務直接互相依賴」——這是指避免同步 HTTP 請求鏈路（例如 Posts → Comments → Query）。但 Event Bus 與各服務之間的**非同步事件通訊**仍然存在，這是正常的。只不過有了 Kubernetes，Event Bus 不需要知道目的地實例的具體位址，只需要發送到一個「服務名稱」。

## 💡 重點摘要

- Docker 解決的是「執行環境一致性」的問題：把程式及其依賴封裝成 Container，隨處可跑。
- Kubernetes 解決的是「多個 Containers 的編排與通訊」問題：自動追蹤服務、動態路由、健康檢查。
- Cluster = 多個 Node（虛擬機）+ 一個 Master（管理協調者）。
- **Service Discovery** 是 Kubernetes 的核心能力：服務只需要知道「對方的名字」，不需要知道 IP/Port。
- 微服務仍然需要互相溝通（透過 Event Bus），只是溝通方式從「點對點硬編址」變成「發到中介層，讓 Kubernetes 自動轉發」。

## 關鍵字

Docker, Container, Image, Kubernetes, Cluster, Node, Master, Service Discovery, Dynamic Scaling, Health Checking, Event Bus, Microservices, YAML Configuration, Asynchronous Communication, Load Balancing
