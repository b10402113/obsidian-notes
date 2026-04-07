# 選擇適合本課程的 Docker 安裝方式

## 📝 課程概述

本單元是 Docker 學習旅程的環境準備篇。老師將詳細說明為何 Docker Desktop 是學習 Docker 的最佳起點，並解釋 Docker 生態系中各種工具的定位，幫助我們在正式進入容器操作前，建立起對「容器工具生態系」的正確認知。

---

## 核心觀念與實作解析

### 為什麼選擇 Docker Desktop？

在 Docker 剛誕生時，市場上真的只有「Docker」這一個工具。但現在情況完全不同了——我們有 OCI（Open Container Initiative）標準、數十種容器 runtime、雲端服務、本地工具等各種選擇。

老師在這裡給出了一個明確的建議：**對於初學者而言，Docker Desktop 是最佳的學習起點。**

為什麼？因為 Docker Desktop 本質上是一個「工具打包器」（tool bundler）：

- 它將十幾種獨立的工具自動化安裝並整合在一起
- 持續維護這些工具的版本更新
- 處理 VPN 相容性、網路配置、本地儲存等繁瑣細節

> **重要區分**：Docker Desktop 是給開發者用的本地工具，**不應該安裝在 Server 上**。Server 環境應該使用 Docker Engine。

---

### Docker Desktop 的授權議題

很多人擔心 Docker Desktop 是否需要付費。老師特別與 Docker 高層確認過：

- **Docker Desktop 對於「學習用途」完全免費**
- 個人使用、小型商業用途也免費
- 只有在企業環境的正式使用才需要付費授權

所以即使你在大型企業上班，只要是用於學習本課程，都可以放心免費使用。

---

### 容器執行的核心原理：Kernel 依賴

這是一個非常關鍵的觀念：**容器必須運行在與其設計相符的 Kernel 上。**

- 如果你為 Windows 建構應用程式，它就必須在 Windows 上運行
- **98-99% 的容器都是設計給 Linux Kernel 的**

這就是為什麼 Docker Desktop 在 Mac 和 Windows 上，會背景執行一個小型的 Linux VM：

| 平台 | Docker Desktop 行為 |
|------|---------------------|
| Mac | 背景啟動輕量級 Linux VM |
| Windows | 透過 WSL2 啟動 Linux 環境 |
| Linux Desktop | 仍然會建立一個小型 VM |

這樣做的好處是：**Docker Desktop 幫我們自動管理這個 VM 的建立、更新、安全修補和刪除。**

---

### 容器工具生態系的三種類型

老師將市面上所有運行容器的方式歸納為三大類：

1. **本地工具（Local Tools）**
   - Docker Desktop、Rancher Desktop、Podman 等
   - 適合學習和本地開發

2. **Server 部署（Server Deployment）**
   - Docker Engine、Kubernetes
   - 適合正式環境

3. **雲端服務（Cloud Services）**
   - AWS Fargate、Google Cloud Run、App Runner 等
   - 不需要直接操作 Docker 或 Kubernetes

> 本課程的前半段會專注在第一類（本地工具），等到需要 Server 或雲端時再介紹其他選項。

---

### OCI 標準化的重要性

Docker 在 2013 年發明了現代容器的三大概念，但這些技術後來都開源了。現在整個產業遵循的是 **OCI（Open Container Initiative）** 標準：

- **容器格式標準化**：任何符合 OCI 的 runtime 都能執行
- **映像檔格式標準化**：用任何工具 build 的 image 都能互通
- **Registry 標準化**：image 的儲存與分發統一介面

這意味著：**你用 Docker build 的 image，可以用 Podman、containerd、nerdctl 等任何 OCI 相容工具來執行。**

---

### 其他本地容器工具（參考用）

雖然本課程以 Docker 為主，但老師也列出了其他選項供參考：

- Rancher Desktop
- Multipass
- Podman
- nerdctl

這些工具各有特色，但對於初學者，建議先熟練 Docker 再探索其他工具。

---

## 💡 重點摘要

- **Docker Desktop 是學習 Docker 的最佳起點，它自動整合並維護十幾種工具，省去繁瑣的環境配置。**
- **Docker Desktop 對學習用途完全免費，即使是企業員工也可以放心用於課程學習。**
- **絕大多數容器都設計給 Linux Kernel，因此 Mac 和 Windows 上需要透過 VM 運行 Linux 環境。**
- **OCI 標準確保了容器生態系的互通性——不同工具 build 的 image 可以跨工具使用。**
- **本課程專注於本地學習環境，Server 和雲端部署會在後續章節介紹。**

---

## 🔑 關鍵字

Docker Desktop, Docker Engine, OCI, Container Runtime, Kernel
