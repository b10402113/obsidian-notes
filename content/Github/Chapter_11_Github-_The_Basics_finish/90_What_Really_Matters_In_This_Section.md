# What Really Matters in This Section — GitHub Basics 總覽

## 📝 課程概述

本節是整個課程的重要轉折點——我們正式從純粹的 Git 指令過渡到 GitHub 的使用。GitHub 是目前全球最大的程式碼代管平台，支援 Git 倉庫的雲端儲存與多人協作。本節涵蓋帳號註冊、SSH 金鑰設定、`git clone`、`git remote` 與 `git push` 等核心指令，讓你能夠將本地倉庫推送到雲端並與他人共享。

## 核心觀念與實作解析

### Git vs. GitHub：最重要的基本區別

這是本節最關鍵的概念，請確保你完全理解：

- **Git**：運行在本地機器的版本控制系統，不需要網路，不需要帳號。你可以在完全不接觸 GitHub 的情況下使用 Git 一輩子。
- **GitHub**：一個代管 Git 倉庫的**網站**，需要網路和帳號才能使用。它為 Git 倉庫提供了雲端備份與多人協作的能力。

> **比喻**：Git 像是你家裡的檔案櫃（本地），GitHub 像是把檔案放到雲端硬碟（遠端），讓你可以從任何地方存取，並與他人共享。

### 為什麼要使用 GitHub？

**1. 雲端備份**
如果本地硬碟損壞或遺失筆電，只要之前有定期 Push 到 GitHub，就能從雲端 Pull 回完整的程式碼與歷史。

**2. 多人協作**
GitHub 讓多位開發者可以：
- 各自 Clone 同一個倉庫到自己的機器
- 分別做開發、提交 Commit
- 將變更 Push 回 GitHub
- 彼此 Pull 對方的新 Commit

**3. 開源專案**
GitHub 是全球開源專案的家園。你可以看到熱門專案（如 React、TensorFlow）的源碼，學習業界最佳實踐，並為開源專案貢獻程式碼。

**4. 求職加分**
雇主常會查看應徵者的 GitHub Profile，了解他們的程式碼風格、專案經驗與貢獻記錄。一個活跃的 GitHub 帳號是很好的技術履歷。

### GitHub 的定價

GitHub 的免費方案對個人開發者來說已經非常充足：
- **無限**的公開與私人倉庫
- **無限**的協作者
- 足以應付大多數個人專案與小型團隊需求

付費方案主要針對企業級功能（如更多的 GitHub Actions 時間、SAML 單一登入等）。

### SSH 金鑰設定：免除每次輸入密碼的麻煩

SSH（Secure Shell）是一種通訊協定，能讓你在與 GitHub 互動時自動完成身份驗證，不需要每次都輸入帳號密碼。設定流程：

1. 檢查是否已有 SSH 金鑰（`~/.ssh/` 目錄）
2. 如果沒有，產生新的 SSH 金鑰
3. 將公鑰（`.pub` 檔案）複製到 GitHub 帳號設定中
4. 之後所有 Push / Pull 操作都會自動通過 SSH 驗證

### 兩種連接本地與 GitHub 的方式

| 方式 | 情境 | 流程 |
|------|------|------|
| **方式一**（現有本地 Repo）| 已有本地 Repo，想放到 GitHub | GitHub 建立空倉庫 → 本地設定 Remote → Push |
| **方式二**（從零開始）| 還沒有任何 Repo，直接想用 GitHub | GitHub 建立空倉庫 → Clone 到本地 → 開始開發 → Push |

### 本節學習地圖

| 指令 | 重要性 | 說明 |
|------|--------|------|
| `git clone <url>` | **Critical** | 從遠端倉庫下載完整 Repo（含歷史）|
| `git remote add origin <url>` | **Critical** | 將本地 Repo 連接到 GitHub 遠端 |
| `git push origin <branch>` | **Critical** | 將本地分支推送到 GitHub |
| SSH 金鑰設定 | **Critical** | 設定身份驗證 |
| GitHub 帳號註冊 | **Critical** | 建立 GitHub 帳號並設定 |

## 💡 重點摘要

- GitHub 是**代管 Git 倉庫的網站**，本身不等於 Git；兩者是完全獨立的技術
- SSH 金鑰讓你不需要每次 Push/Pull 時都輸入帳號密碼
- 有兩種連接方式：**方式一**（本地已有 Repo → Push 上 GitHub）vs **方式二**（先在 GitHub 建立空 Repo → Clone 下來）
- GitHub 的免費方案功能完整，足以應付個人開發者的所有需求
- GitHub Profile 是技術人的履歷，參與開源專案是很好的學習與展示方式

## 關鍵字

GitHub, Git, SSH, SSH key, git clone, git remote, git push, origin, cloud backup, collaboration, open source, GitHub Profile
