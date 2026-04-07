# VS Code DevOps 開發環境設定指南

## 📝 課程概述

本單元介紹為什麼 Visual Studio Code 是 DevOps 和容器開發的最佳編輯器選擇。老師將分享他從 VIM 轉換到 VS Code 的心得，並推薦本課程必備的 extension，幫助我們在撰寫 Dockerfile、Kubernetes YAML 和 Docker Compose 設定檔時更加得心應手。

---

## 核心觀念與實作解析

### 為什麼選擇 VS Code？

老師分享了他長達數十年的編輯器使用經驗：

- **VIM 時代**：作為 sysadmin，需要在 Server 上編輯檔案，VIM 是標準選擇
- **嘗試過其他編輯器**：Atom、各種付費編輯器，但都沒有長期使用
- **VS Code 出現後**：功能強大、免費、跨平台，成為最終選擇

**VS Code 對 DevOps 的優勢：**

- 語法高亮（Syntax Highlighting）
- 即時錯誤提示（在你輸入時就會檢查語法是否正確）
- 豐富的 extension 生態系
- 跨平台支援（Windows、Mac、Linux）

> 本課程不需要寫程式，但 VS Code 對於編輯 Dockerfile、Compose YAML、Kubernetes 設定檔等非常有幫助。

---

### 取得 VS Code

有多種方式可以使用 VS Code：

| 方式 | 說明 |
|------|------|
| 官方網站下載 | [code.visualstudio.com](https://code.visualstudio.com) |
| Microsoft Store | Windows 使用者可從市集安裝 |
| 網頁版 | [vscode.dev](https://vscode.dev) — 在瀏覽器中執行完整功能 |
| GitHub 整合 | [github.dev](https://github.dev) — 直接在 GitHub repository 中編輯 |

---

### 設定同步（Settings Sync）

**重要第一步：登入並啟用設定同步。**

VS Code 可以將你的設定、extension、快捷鍵等同步到雲端：

- 點擊左下角的帳號圖示
- 登入 Microsoft 或 GitHub 帳號
- 啟用 Settings Sync

**好處：**

- 不需要手動備份設定
- 換電腦時自動同步所有配置
- 多台電腦保持一致的開發環境

---

### 開啟課程 Repository

當你用 VS Code 開啟課程 repository 時，會看到一個彈出視窗提示安裝推薦的 extension。

**這是怎麼運作的？**

課程 repository 中有一個 `.vscode/extensions.json` 檔案，裡面列出了建議安裝的 extension。VS Code 會自動讀取並提示你安裝。

你可以選擇：

- 點擊 "Install" 一次性安裝所有推薦 extension
- 或點擊 "Show Recommendations" 查看詳細清單

---

### 必備 Extension 推薦

#### 1. Docker Extension（官方）

**功能：**

- Dockerfile 和 Docker Compose 檔案的語法高亮
- 自動完成（IntelliSense）
- 語法錯誤檢查
- 直接在 VS Code 中管理 container、image、network、volume

> 老師提到，在這個課程剛創建時（當時還沒有 Docker extension），學員需要花大量時間查閱 Docker 文件。現在有了 extension，可以在輸入時就看到錯誤，大幅提升學習效率。

#### 2. Kubernetes Extension（官方）

**功能：**

- Kubernetes YAML 檔案的語法高亮和驗證
- 內建 snippets（程式碼片段）
- 直接在 VS Code 中管理 Kubernetes 資源

**辨識官方 extension：**

認明 Microsoft 藍色勾勾標誌，代表這是 Microsoft 官方維護的安全 extension。

#### 3. Remote Development Extension Pack

這是老師每週都會使用的 extension，包含三個子 extension：

**Remote - SSH：**

- 透過 SSH 連線到遠端 Server
- 在 VS Code 中直接編輯遠端檔案
- 就像在本地開發一樣，但所有指令都在遠端執行

**Remote - WSL：**

- 讓 Windows + WSL 使用者無縫整合
- VS Code 可以直接存取 WSL 檔案系統
- 編輯 WSL 中的檔案就像編輯本地檔案

**Remote - Containers：**

- 在 container 中開發
- 可以將整個開發環境打包成 container

**使用方式：**

安裝後，左下角會出現一個 `><` 圖示，點擊後可以選擇要連線的遠端環境。

#### 4. Live Share（選用）

**用途：**

- 即時協作編輯（類似 Google Docs）
- 適合 pair programming、code review、線上教學

**特點：**

- 使用 Microsoft 雲端建立安全 tunnel
- 雙方可以同時編輯、看到對方的游標位置
- 適合團隊合作或與朋友一起學習

---

### VS Code 的智慧推薦

即使你沒有預先安裝 extension，VS Code 也會根據情境推薦：

- 開啟 Dockerfile → 提示安裝 Docker extension
- 開啟 Terraform 檔案 → 提示安裝 Terraform extension
- 偵測到 Docker 已安裝 → 提示安裝相關 extension

> 所以不用擔心要一次安裝所有 extension，VS Code 會在需要時提醒你。

---

## 💡 重點摘要

- **VS Code 是 DevOps 工作的最佳編輯器，提供語法高亮、錯誤檢查和豐富的 extension 生態系。**
- **啟用 Settings Sync 可以跨裝置同步所有設定，換電腦時無需手動備份。**
- **Docker 和 Kubernetes 官方 extension 能大幅提升撰寫 Dockerfile 和 YAML 的效率。**
- **Remote Development extension 讓你可以像本地開發一樣操作遠端 Server 或 WSL 環境。**
- **Live Share 是團隊協作和線上教學的利器，可以即時共享程式碼編輯畫面。**

---

## 🔑 關鍵字

VS Code, Docker Extension, Kubernetes Extension, Remote Development, Live Share
