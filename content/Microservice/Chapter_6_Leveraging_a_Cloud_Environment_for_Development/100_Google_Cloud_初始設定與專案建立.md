# Google Cloud 初始設定與專案建立

## 📝 課程概述

本單元帶領學員完成 Google Cloud 的帳號申請與第一個專案（Project）的建立，並介紹 **Google Cloud SDK** 的安裝。我們將理解 `kubectl` 如何透過 **Context** 機制連線到不同的 Kubernetes 叢集，為後續設定遠端叢集奠定基礎。

---

## 核心觀念與實作解析

### 申請 Google Cloud 帳戶

1. 前往 `cloud.google.com`（可用 URL 中的 `/free` 結尾取得免費額度說明）
2. 若未登入，點擊 **Sign up for new account**；若已登入，則點 **Go to console**
3. 選擇所在地區、填入帳單資訊，並選擇身份（個人或企業）
4. **需要信用卡**才能完成申請，即使我們後續會使用免費額度

> 新帳戶自動獲得 **$300 美元免費額度**，強烈建議建立新帳戶以確保享有優惠。

---

### 建立第一個 Google Cloud Project

1. 進入 Google Cloud Console 後，點擊左上角的專案下拉選單
2. 點擊 **New Project**
3. 輸入專案名稱（例如 `ticketing-dev`）
4. 點擊 Create

**重要觀念**：專案建立**並非瞬間完成**，約需 1-2 分鐘。頁面右上角會出現通知，告知你專案已建立完成後，再點擊切換進入該專案。

進入後，確認左上角的專案選擇器已顯示 `ticketing-dev`，代表你目前在該專案的上下文中操作。

---

### kubectl 與 Context 的關係

理解 `kubectl` 如何選擇要連線的叢集，是設定遠端開發環境的核心概念：

**Context** 是 `kubectl` 用來儲存叢集連線資訊的設定，包含：
- **授權憑證（credentials）**
- **使用者資訊**
- **API Server 的 IP 位址**

**本地 Context**：當你首次安裝 Docker Desktop 時，會自動建立一個名為 `docker-desktop` 的 Context，專門用來連線本地叢集。你可以透過 Docker Desktop 圖示 → Kubernetes 查看目前使用的 Context。

**我們需要做什麼**：再新增一個 Context，用來連線剛才在 Google Cloud 建立的遠端叢集。

---

### 新增 Context 的兩種方式

| 方式 | 說明 |
|------|------|
| 手動複製設定檔 | 在 Google Cloud Console 中找到設定檔內容，手動貼到本機 `~/.kube/config` |
| 安裝 Google Cloud SDK（推薦） | SDK 會**自動管理 Context**，並自動偵測你在 GCP 上建立的叢集 |

老師建議使用 **Google Cloud SDK**（即 `gcloud` 命令列工具），因為它能自動同步、更新 Context，省去手動管理的麻煩。

---

### 安裝 Google Cloud SDK

1. 前往 `cloud.google.com/sdk`（若連結失效，Google 搜尋「Google Cloud SDK」即可找到）
2. 選擇你的作業系統（macOS / Windows / Linux）
3. 按照指示完成安裝

**特別注意**：**不要**在此時執行 `gcloud init` 初始化命令，這個步驟會在下一個影片中與老師一起完成。

安裝完成後，只需要確認 SDK 已就緒即可，Context 的設定留待後續處理。

---

## 💡 重點摘要

- **申請 Google Cloud 需要信用卡，新帳戶自動獲得 $300 免費額度。**
- **Google Cloud 的所有資源（叢集、Image Repo 等）都隸屬於一個 Project，建立 Project 是後續所有操作的起點。**
- **kubectl 透過 Context 決定連線到哪個叢集，本地 Docker Desktop 與遠端 GKE 各有自己的 Context。**
- **Google Cloud SDK（gcloud）會自動管理 kubectl Context，大幅簡化設定流程。**
- **在 SDK 安裝完成後，先不要執行 `gcloud init`，該步驟留待下一階段處理。**

---

## 🔑 關鍵字

Google Cloud Console, Project, kubectl Context, gcloud SDK, Docker Desktop, Credential
