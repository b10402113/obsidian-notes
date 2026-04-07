# Linux Desktop 環境 Docker Desktop 安裝指南

## 📝 課程概述

本單元專為 Linux Desktop 使用者設計。Docker Desktop for Linux 是 2022 年的新產品，終於讓 Mac、Windows、Linux 三大平台都能使用一致的 Docker Desktop 體驗。本課將說明安裝步驟、與傳統 Docker Engine 的差異，以及重要的注意事項。

---

## 核心觀念與實作解析

### Docker Desktop vs. Docker Engine

在 Docker Desktop for Linux 出現之前，Linux 使用者只能安裝原生的 Docker Engine。兩者有本質上的差異：

| 特性 | Docker Engine | Docker Desktop |
|------|---------------|----------------|
| 安裝方式 | 套件管理器（apt/yum） | 官方安裝包 |
| 執行環境 | 直接在 host kernel | 在背景 VM 中 |
| GUI | 無 | 有 Dashboard |
| 更新維護 | 手動 | 自動更新 |
| 一鍵重置 | 無 | 有（Reset 功能） |

**Docker Desktop 是 Docker Engine 的超集合（superset）**，包含了 Engine、CLI、以及許多額外工具。

---

### 為什麼 Docker Desktop 在 Linux 上也使用 VM？

這是一個有趣的設計決策。即使 Linux 本身就有原生 Kernel，Docker Desktop 仍然選擇建立一個小型 VM。原因包括：

1. **跨平台一致性**：Mac、Windows、Linux 三個平台有相同的執行環境
2. **隔離性**：Container 在 VM 中運行，與 host 系統隔離
3. **可重置性**：可以一鍵刪除所有變更，恢復初始狀態
4. **自動更新**：Docker Desktop 統一管理所有工具的版本

> 你可以同時安裝 Docker Desktop 和 Docker Engine，並透過 `docker context` 指令在兩者之間切換。但對於初學者，建議專注於 Docker Desktop 即可。

---

### 支援的 Linux 發行版

**重要限制**：Docker Desktop 並非支援所有 Linux 發行版。

老師錄製課程時（以 Ubuntu 22.04 為例）：

- ✅ **Ubuntu 22.04**：完全支援
- ❌ **Linux Mint**：當時不支援（Docker Desktop 需要 GNOME 相關元件）

**建議做法：**

1. 前往 Docker 官方文件確認你的發行版是否支援
2. 支援清單會隨時間擴充，請以官方文件為準

---

### 安裝前置需求

Docker Desktop for Linux 的安裝比 Mac/Windows 複雜許多，需要先安裝大量依賴套件。

以 Ubuntu 22.04 為例，需要執行一系列前置安裝：

```bash
# 安裝基本工具
sudo apt-get update
sudo apt-get install curl

# 安裝 Docker Desktop 所需的大量依賴套件
# （詳細清單請參考官方文件，會隨版本更新）
```

> 這正是 Docker Desktop 不支援所有發行版的原因——依賴太多特定版本的系統函式庫。

---

### 安裝步驟

#### Step 1：安裝依賴套件

按照官方文件的指示，使用 `apt-get` 安裝所有必要的依賴。

#### Step 2：下載並安裝 Docker Desktop

從 Docker 官方文件下載對應的安裝包，使用套件管理器安裝。

#### Step 3：啟動 Docker Desktop

安裝完成後，在你的應用程式選單中搜尋 "Docker" 並啟動。

#### Step 4：接受授權

首次啟動會顯示授權協議。**學習用途完全免費**，放心接受即可。

---

### 登入 Docker Hub 的額外步驟

在 Linux 上登入 Docker Hub 可能會遇到 "Credential store not initialized" 錯誤。這是因為需要設定憑證儲存機制。

**解決方式：**

1. 安裝並設定 GPG key
2. 使用 `pass` 工具來管理憑證

```bash
# 建立 GPG key
gpg --gen-key

# 設定 pass 使用這個 key
pass init <your-gpg-id>
```

完成後，再次嘗試在 Docker Desktop 中登入，應該就能成功顯示登入畫面。

---

### 重要設定調整

點擊系統匣的 Moby 圖示 → Settings，調整以下設定：

#### Resources（資源配置）

Docker 預設值較為保守。老師建議：

- **CPU**：根據你的硬體調高
- **Memory**：建議 6-8 GB 或更高

> Docker 不使用時會自動釋放資源，所以不用擔心佔用問題。

#### File Sharing（檔案共享）

Docker Desktop 需要知道哪些目錄可以共享給 container。

**最佳實踐：**

- 將程式碼放在你的 home 目錄（`/home/username/`）
- home 目錄預設已包含在共享清單中
- 建議建立一個 `~/code` 目錄來存放所有專案

---

### 驗證安裝

在終端機中執行：

```bash
docker version
```

成功時會顯示 Client 和 Server 版本資訊。

---

### Visual Studio Code 設定

VS Code 是老師推薦的編輯器，特點包括：

- 支援 Dockerfile、Kubernetes YAML 等檔案格式
- 會自動推薦相關 extension（例如：開啟 Terraform 檔案時推薦 Terraform extension）
- 跨平台，設定可同步

---

### 取得課程 Repository

從課程資源的連結前往 GitHub，clone 課程範例：

```bash
cd ~
git clone <repository URL>
```

建議放在 home 目錄下，權限設定最單純。

---

## 💡 重點摘要

- **Docker Desktop for Linux 是 2022 年的新產品，讓 Linux Desktop 也能享有與 Mac/Windows 一致的體驗。**
- **Docker Desktop 在 Linux 上仍會建立一個小型 VM，以提供隔離性、可重置性和自動更新。**
- **並非所有 Linux 發行版都支援，安裝前請確認官方支援清單；Ubuntu 22.04 是最佳選擇。**
- **登入 Docker Hub 時若遇到憑證錯誤，需要設定 GPG key 和 pass 工具。**
- **將程式碼放在 home 目錄下，確保 Docker Desktop 的檔案共享功能正常運作。**

---

## 🔑 關鍵字

Docker Desktop, Docker Engine, Linux Desktop, GNOME, GPG
