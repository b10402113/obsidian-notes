# Docker 與 Kubernetes：微服務架構中的角色

## 📝 課程概述

本單元介紹 Docker 與 Kubernetes 在微服務部署脈絡中的定位。我們會看到 Docker 如何透過 Container 解決「環境一致性」的問題，以及 Kubernetes 如何作為一個「服務協調層」，自動處理服務之間的溝通與動態擴縮容——徹底告別前一章提到的 Event Bus 耦合噩夢。

---

## 核心觀念與實作解析

### Docker 的核心價值：隔離與封裝

Docker 將「執行一個程式」所需的所有依賴（runtime、工具鏈、配置）全部打包進一個名為 **Container** 的隔離執行環境。對比本機直接執行：

| 現況（本機直接執行） | 使用 Docker Container |
|---|---|
| 依賴本機已安裝的 Node.js 版本 | Container 內建精確的 Node 版本 |
| `npm start` 是隱性啟動知識 | Container 的 `CMD` 明確宣告啟動方式 |
| 跨機器環境差異導致「在我機子上能跑」 | Container 一次封裝，到哪都一樣 |

> **Container 與 Image 的關係**：Image 是「只讀的模板」（類比 Class），Container 是「根據模板生出來的運行實例」（類比 Object）。一個 Image 可以同時啟動多個 Container。

### Kubernetes 的核心價值：協調與抽象

Kubernetes（簡稱 K8s）是一套**容器編排系統**。它的核心職責是：

1. **接收配置文件**（宣告式）：你告訴它「我想要 2 份 Posts 服務」而非「幫我啟動 Container A，然後再啟動 Container B」
2. **自動創建與調度 Container**：將這些 Container 隨機分配到 Cluster 內的各個節點（Node，即 VM）上執行
3. **自動處理服務間的網路通訊**：透過一個內建的「虛擬網路層」，讓各服務只須知道「目標服務的名稱」而非 IP + Port

### 為什麼需要 Kubernetes？

回到前一章的問題：當 Comments 服務需要擴展到多個副本時，Event Bus 不需要知道「這些副本在哪些 IP」。

在 Kubernetes 環境中，Event Bus 只需把請求發給一個固定的「服務名」（例如 `comments`），Kubernetes 的 **Service 層**會自動將請求轉發到所有運行中的 Comments Pod（Container）。你可以任意增減副本數量，Event Bus 的程式碼**完全不需要改動**。

```
Event Bus  ──發送到 "comments"──►  Kubernetes Service
                                          │
                        ┌─────────────────┼─────────────────┐
                        ▼                 ▼                 ▼
                   Comments Pod 1   Comments Pod 2   Comments Pod 3
```

這就是**宣告式配置**與**第三人協調**的威力——Event Bus 不再需要維護任何動態狀態。

### Cluster 架構簡述

- **Cluster**：一組節點（Node）的集合，類比一個「運算機房」
- **Node**：實際執行 Container 的虛擬機器（或實體機），負責承載工作負載
- **Master**：Cluster 的大腦，負責讀取配置文件、調度 Container、維護整體狀態

### 微服務間的通訊：為何還是需要通訊？

「微服務不該直接互相呼叫」指的是**同步的跨服務請求鏈**（例如串聯依賴），這會造成緊密耦合。但微服務架構中，**非同步的事件匯流排通訊**仍然存在——每個服務都需要將事件發送到同一個通訊管道（Event Bus），也需要與自己專屬的資料庫溝通。Kubernetes 的 Service 層讓這一切變得自動且透明。

---

## 💡 重點摘要

- **Docker Container 封裝了執行環境與啟動命令，讓任何程式在何處都能以相同方式運行。**
- **Kubernetes 透過宣告式配置，自動處理 Container 的創建、調度與通訊——應用程式不再需要知道服務的 IP 與 Port。**
- Kubernetes 的 Service 元件提供了一個「固定服務名」作為通訊入口，背後自動負載平衡到所有運行中的 Pod。
- Container 與 Pod 的關係：Pod 是 Kubernetes 的最小調度單位，通常由一個或一組 Container 組成。
- Docker + Kubernetes 的組合是微服務部署的黃金標準：Docker 解決「環境封裝」，K8s 解決「動態協調」。

---

## 🔑 關鍵字

Docker, Container, Image, Kubernetes, Cluster, Node, Master, Service, Pod
