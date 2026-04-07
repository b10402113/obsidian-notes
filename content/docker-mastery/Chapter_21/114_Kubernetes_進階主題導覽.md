# Kubernetes 進階主題導覽

## 📝 課程概述

本單元是進階章節的開場介紹，在學完 Kubernetes 的基礎概念（Deployment、ReplicaSet、Service）後，講師將帶領學員認識 Kubernetes 生態系中更廣泛的工具與技術。這個章節不會深入指令操作，而是著重於「讓你知道有哪些重要工具值得關注」，為後續的深入學習建立正確的認知地圖。

---

## 核心觀念與實作解析

### Kubernetes 生態系的廣度

Kubernetes 的生態系極其龐大，每天都有新專案誕生。本課程截至目前為止，其實只觸及了 Kubernetes 生態的表面。這個章節的目的是讓學員具備「全局視野」，了解在實務上—even 是最小的三節點叢集—你可能會需要關注哪些工具。

> 這個章節比較「發散」，但這正是 Kubernetes 生態的真實寫照。你可以花 100 小時討論 Kubernetes，仍然無法涵蓋所有主題。

---

### Storage 與 StatefulSet

在 Kubernetes 中，我們始終假設 Container 是 **stateless（無狀態）** 的。當你使用 `kubectl apply` 更新應用時，舊的 Container 會被銷毀，裡面的所有資料也會隨之消失。這與 Docker、Compose、Swarm 的概念一致。

但如果你的應用需要保存資料（例如資料庫），Kubernetes 提供了幾種解決方案：

#### 標準 Volume

與 Docker 和 Swarm 類似，你可以在 Pod 的 YAML 檔案中，於 `spec` 區塊內加入 `volume` 語句。這個 Volume 技術上是綁定在 **Pod 層級**，但可以連接到 Pod 內的一個或多個 Container。

#### Persistent Volume 與 Persistent Volume Claim

這是更進階的儲存方案，核心概念是 **將 Volume 的定義與 Pod 分離**：

- **Persistent Volume（PV）**：由儲存團隊或第三方儲存服務商建立的儲存資源
- **Persistent Volume Claim（PVC）**：應用部署者在 Pod spec 中宣告「我需要什麼類型的儲存」

> **Why 這樣設計？** 在企業環境中，管理儲存的團隊往往與管理 Kubernetes 的團隊不同。這種分離讓儲存團隊可以專注於管理 PV，而應用開發者只需透過 PVC 宣告需求即可。

#### StatefulSet

StatefulSet 是 Kubernetes 中專門為 **有狀態應用**（如資料庫）設計的資源類型，與 Deployment 不同的是：

- 保持固定的 Pod 名稱與 IP 位址
- 更適合需要穩定身分的應用

> **講師建議**：如果你的第一個 Kubernetes 部署涉及大量 StatefulSet，請三思。理想情況下，初次部署應以 **stateless 應用** 為主（如 Web frontend、API、Worker），並盡可能使用 **雲端代管資料庫**。資料庫與持久化儲存會大幅增加複雜度與測試成本。

---

### CSI（Container Storage Interface）

早期 Kubernetes 將所有第三方儲存驅動程式 **in-tree**（內建於 Kubernetes 二進位檔中），這帶來了幾個問題：

- 儲存供應商必須配合 Kubernetes 的發布週期
- 每個 Kubernetes 版本都內建了大量你可能根本用不到的儲存驅動

為了解決這個問題，業界制定了 **CSI（Container Storage Interface）** 標準：

- 儲存供應商可以獨立開發 **Plugin**
- Plugin 以標準 API 運作，與 Kubernetes 核心解耦
- 更靈活、更新週期不受 Kubernetes 限制

> CSI 是 Kubernetes 儲存的未來方向，但部分儲存供應商的 CSI 支援仍在成熟中，某些情況下可能仍需使用傳統的內建 Plugin。

---

## 💡 重點摘要

- **Kubernetes 生態極其龐大，本課程只涵蓋基礎；這個章節目標是建立全局視野，讓你知道有哪些重要工具值得關注。**
- **StatefulSet 專為有狀態應用設計，但初次部署應以 stateless 為主，盡量使用雲端代管資料庫以降低複雜度。**
- **Persistent Volume 與 Claim 的設計將儲存管理與應用部署職責分離，適合企業環境的分工模式。**
- **CSI 是儲存驅動的標準化介面，讓第三方儲存供應商可以獨立開發 Plugin，不再受限於 Kubernetes 發布週期。**

---

## 🔑 關鍵字

StatefulSet, Persistent Volume, Persistent Volume Claim, CSI, Volume
