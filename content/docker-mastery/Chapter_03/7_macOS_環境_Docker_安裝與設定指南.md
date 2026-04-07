# macOS 環境 Docker 安裝與設定指南

## 📝 課程概述

本單元專為 macOS 使用者設計，說明如何在 Mac 上安裝 Docker Desktop、針對 Intel 與 Apple Silicon（M1/M2）晶片的注意事項、效能優化設定，以及課程所需的環境準備。

---

## 核心觀念與實作解析

### macOS 與 Docker 的相容性

Docker Desktop 在 Mac 上的運作相對單純，不像 Windows 需要處理 BIOS 或虛擬化設定。**只要你的 Mac 是近 5-7 年內的機型，且運行最近 2-3 個主要版本的 macOS，通常都能順利運行。**

### Intel Mac vs. Apple Silicon Mac

下載 Docker Desktop 時，請務必選擇正確的版本：

| Mac 類型 | 處理器 | 下載版本 |
|----------|--------|----------|
| Intel Mac | Intel Core 系列 | Intel 版本 |
| Apple Silicon | M1/M2/M3 系列 | Apple Silicon 版本 |

> 下載錯誤版本將無法正常運行。Docker 官網會自動偵測你的架構，但仍建議手動確認。

---

### Docker Desktop 的運作原理

Mac 的 Kernel 是基於 BSD 的 XNU，與 Linux Kernel 不相容。因此 Docker Desktop 必須：

1. **在背景啟動一個輕量級 Linux VM**
2. 在 VM 中運行 Docker Engine
3. 透過虛擬化技術與 macOS 整合

這個 VM 由 Docker Desktop 自動管理，具有以下特性：

- 低能耗設計
- 可以隨時暫停（Pause）
- 啟動時需要一些時間（Dashboard 下方會顯示橘色進度條）

---

### 安裝步驟

#### Step 1：下載 Docker Desktop

前往 Docker 官方網站下載 DMG 檔案，這是標準的 Mac 應用程式安裝方式。

#### Step 2：安裝應用程式

將下載的 Docker app 拖曳到 Applications 資料夾即可完成安裝。

#### Step 3：接受授權協議

首次啟動時會顯示 EULA。**學習用途完全免費**，點擊 "I accept" 繼續。

#### Step 4：等待啟動完成

Docker Desktop 啟動時：

1. 選單列會出現鯨魚圖示（Moby）
2. Dashboard 下方會從橘色變為綠色
3. 綠色表示 Docker Engine 已準備就緒

---

### 重要設定調整

點擊選單列的 Moby 圖示 → 選擇 Preferences（偏好設定），調整以下設定：

#### Resources（資源配置）

Docker 預設的資源配置較為保守。當你開始運行多容器工作負載時，可能需要調高：

- **Memory（記憶體）**：建議至少分配總記憶體的一半
- **Disk（磁碟空間）**：建議設定 100-200 GB

> Docker 採用動態分配，不會預先佔用這些資源。只有在實際使用時才會消耗，未使用時會自動釋放。磁碟空間也會在重啟時自動壓縮。

#### Experimental Features（實驗性功能）

老師建議啟用以下選項以提升效能：

- **新虛擬化支援**：使用 macOS 最新的虛擬化框架，比舊版更快
- **VirtioFS**：加速檔案共享，特別是在 bind mount 時效果顯著

> VirtioFS 是 2022 年引入的新功能，大幅改善了 host 與 container 之間的檔案存取速度。

---

### 登入 Docker Hub

與 Windows 環境相同，老師強烈建議登入 Docker Hub：

**Pull 限制說明：**

| 狀態 | 限制 |
|------|------|
| 未登入 | ~100 pulls / 6 小時 |
| 登入免費帳號 | ~200 pulls / 6 小時 |

**操作方式：**

1. 在選單列點擊 Moby 圖示 → Sign in
2. 或在 Dashboard 中點擊 Sign in
3. 輸入你的 Docker ID（不是 email）和密碼

> 在公司網路環境中，所有人可能共用同一個對外 IP，這會加速消耗 pull 配額。登入後可以有效避免這個問題。

---

### 終端機環境

macOS 內建的 Terminal.app 或 iTerm2 都可以直接使用，不需要額外設定。Mac 原生就支援：

- bash
- zsh（現代 macOS 預設）

這與 Linux 的 shell 完全相容，本課程的所有指令都可以直接執行。

---

### 驗證安裝

在終端機中執行：

```bash
docker version
```

成功時會顯示：

- **Client** 版本（你的 Docker CLI）
- **Server** 版本（VM 中的 Docker Engine）

---

### Visual Studio Code

老師推薦使用 VS Code 作為編輯器，原因如下：

- 支援 Dockerfile、Docker Compose、Kubernetes YAML 的語法高亮
- 豐富的 extension 生態系
- 跨平台，可以同步設定到其他電腦

> 本課程不需要寫程式，但 VS Code 對於編輯 DevOps 相關的設定檔非常有幫助。

---

### 取得課程 Repository

從課程資源的連結前往 GitHub，使用 git clone 下載課程範例檔案：

```bash
cd ~
git clone <repository URL>
```

建議將 repository 放在你的使用者目錄下（`/Users/username/` 或 `~/`），這樣權限設定最單純。

---

## 💡 重點摘要

- **下載 Docker Desktop 時請確認選擇正確版本：Intel Mac 選 Intel 版本，Apple Silicon 選 M1 版本。**
- **Docker Desktop 在 Mac 上運行一個輕量級 Linux VM，自動管理虛擬化細節。**
- **建議調高 Resources 中的 Memory 和 Disk 設定，並啟用實驗性的 VirtioFS 以提升檔案存取效能。**
- **登入 Docker Hub 免費帳號可獲得雙倍的 image pull 次數。**
- **macOS 內建的 Terminal 和 bash/zsh 與 Linux 完全相容，可以直接執行本課程所有指令。**

---

## 🔑 關鍵字

Docker Desktop, Apple Silicon, Intel Mac, VirtioFS, Docker Hub
