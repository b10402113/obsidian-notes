# Windows 環境 Docker 安裝與設定指南

## 📝 課程概述

本單元專為 Windows 10/11 使用者設計，詳細說明如何在 Windows 環境下正確安裝 Docker Desktop、配置 WSL2（Windows Subsystem for Linux 2）、設定最佳化的開發環境，以及避免課程中可能遇到的常見問題。

---

## 核心觀念與實作解析

### Windows 環境需求

在開始之前，請確認你的環境符合以下條件：

- **作業系統**：Windows 10 或 Windows 11（任何版本：Home、Pro、Enterprise 都可以）
- **不需要 Hyper-V**：現代 Docker Desktop 使用 WSL2，比舊版 Hyper-V 更省電、更高效

> 老師特別推薦使用 Visual Studio Code 作為編輯器，它對 DevOps 和 Docker 有極佳的支援。

---

### WSL2：Docker 在 Windows 上的核心技術

WSL2（Windows Subsystem for Linux 2）是 Microsoft 官方內建的功能，它是目前**在 Windows 上運行 Linux 的最佳方式**。

**為什麼 Docker 需要WSL2？**

因為容器需要 Linux Kernel 才能運作。Windows 不直接提供 Linux Kernel，所以 Docker Desktop 必須：

1. 在背景啟動一個輕量級 Linux VM
2. 在這個 VM 中運行 Docker Engine
3. 透過 WSL2 與 Windows host 整合

**WSL2 vs. Hyper-V 的差異：**

| 特性 | Hyper-V（舊版） | WSL2（新版） |
|------|----------------|--------------|
| 資源使用 | 較重 | 較輕 |
| 電池消耗 | 較高 | 較低 |
| 啟動速度 | 較慢 | 較快 |
| 整合性 | 一般 | 優秀 |

---

### 安裝步驟詳解

#### Step 1：下載與安裝 Docker Desktop

前往 Docker 官方文件，下載 Windows 版本的安裝程式。安裝過程中會看到詢問是否使用 WSL2 的選項，**請務必選擇 WSL2**。

#### Step 2：處理必要的系統更新

安裝過程中可能需要：

1. **安裝 WSL2** — 可能需要重新啟動電腦
2. **更新 Linux Kernel** — Windows 會額外執行一個更新程式

> Windows 將 Linux Kernel 更新獨立於 Windows Update 之外，這樣可以讓你單獨更新 Linux Kernel 而不需要更新整個 Windows 系統。

#### Step 3：接受授權協議

Docker Desktop 會顯示 EULA（End User License Agreement）。重點是：**學習用途完全免費**，你可以放心點擊 "I accept"。

---

### 登入 Docker Hub 的重要性

當 Docker Desktop 成功啟動後，你會看到 Dashboard 介面。老師強烈建議立即登入 Docker Hub 帳號：

**為什麼要登入？**

Docker Hub 有 pull（下載）限制：

| 狀態 | Pull 限制 |
|------|-----------|
| 未登入 | 約 100 次 / 6 小時 |
| 登入免費帳號 | 約 200 次 / 6 小時 |

> 如果你在公司網路環境，公司所有人可能共用同一個對外 IP，這個限制會更快用完。登入後可以獲得雙倍的 pull 次數。

**操作方式：**

1. 前往 hub.docker.com 註冊免費帳號
2. 在 Docker Desktop 中登入你的 Docker ID

---

### 常見錯誤排除：CPU虛擬化未啟用

如果你在啟動 Docker 時遇到虛擬化相關錯誤，這表示你的 **BIOS 中未啟用 CPU 虛擬化功能**。

**如何解決：**

1. 重新啟動電腦進入 BIOS 設定
2. 尋找與虛擬化相關的選項（名稱可能是：VT-x、Intel Virtualization Technology、AMD-V 等）
3. 啟用該功能
4. 儲存設定並重新啟動

> 如果你之前使用過 VirtualBox、VMware、Parallels 等虛擬機軟體，這個功能應該已經啟用了。

---

### Docker Desktop 設定調整

安裝完成後，老師建議調整以下設定：

#### 存取設定的方式

1. 在系統匣（右下角）找到 Moby 圖示（Docker 的吉祥物鯨魚）
2. 右鍵點擊 → 選擇 Settings

#### 重要設定項目

**1. WSL Integration（WSL 整合）**

如果你安裝了多個 Linux 發行版（如 Ubuntu、Debian、Kali），需要在這裡啟用 Docker 支援：

- 進入 Settings → Resources → WSL Integration
- 將你要使用的 Linux 發行版開關切換為藍色（啟用）

**2. Resources（資源配置）**

當你開始運行多容器工作負載時，可能需要調整：

- CPU 核心數
- 記憶體大小
- 磁碟空間

---

### 安裝 Ubuntu WSL 發行版

老師建議安裝 Ubuntu 作為主要的 WSL 環境：

**為什麼選擇 Ubuntu？**

- 最廣泛使用的 Linux 發行版
- Docker 官方支援度最高
- 網路上教學資源最豐富

**安裝步驟：**

1. 開啟 Microsoft Store
2. 搜尋 "Ubuntu"
3. 下載並安裝 Ubuntu WSL 版本
4. 首次啟動時，設定一組使用者名稱和密碼

> 這個 Linux 帳號與你的 Windows 帳號是分開的，請務必記住你設定的帳密。

---

### 使用 Windows Terminal 作為預設終端機

Microsoft 現在推薦使用 **Windows Terminal** 取代舊版的 PowerShell 和 CMD。

**Windows Terminal 的優點：**

- 支援多種 Shell：PowerShell、CMD、WSL
- 可以自訂字體、顏色、主題
- 自動列出你安裝的所有 WSL 發行版

**設定建議：**

老師建議將預設設定改為：

1. 預設 Shell → Ubuntu（bash）
2. 預設終端機應用程式 → Windows Terminal

這樣每次開啟終端機時，就會直接進入 bash 環境，與 Mac/Linux 使用者的指令相容性最高。

---

### 驗證安裝成功

在終端機中執行以下指令：

```bash
docker version
```

如果安裝成功，你應該會看到：

- **Client** 版本資訊
- **Server** 版本資訊

這表示你的 Docker CLI 可以正確與 Docker Engine（在 WSL2 VM 中）溝通。

---

### 課程 Repository 的放置位置

老師特別提醒：**將課程 repository clone 到 WSL 的檔案系統中，而不是 Windows 的 C:\ 路徑。**

**如何判斷？**

- ❌ 錯誤：路徑開頭是 `C:\Users\...`（Windows host 檔案系統）
- ✅ 正確：路徑開頭是 `/home/username/...`（WSL Linux 檔案系統）

**為什麼？**

因為 Docker Engine 運行在 WSL 的 Linux 環境中，在 Linux 檔案系統中操作會更快速、更順暢。

**操作方式：**

```bash
# 在 WSL Ubuntu 終端機中
cd ~
git clone <課程 repository URL>
```

---

## 💡 重點摘要

- **Docker Desktop 在 Windows 上依賴 WSL2 運行 Linux 環境，比舊版 Hyper-V 更高效省電。**
- **登入 Docker Hub 免費帳號可以獲得雙倍的 image pull 次數，避免課程中遇到下載限制。**
- **如果遇到虛擬化錯誤，請進入 BIOS 啟用 CPU 虛擬化功能（VT-x/AMD-V）。**
- **安裝 Ubuntu WSL 發行版，並使用 Windows Terminal 作為預設終端機，確保指令與 Mac/Linux 相容。**
- **將課程 repository clone 到 WSL 檔案系統（/home/...）而非 Windows 檔案系統（C:\...）。**

---

## 🔑 關鍵字

WSL2, Docker Desktop, Windows Terminal, Ubuntu, Virtualization
