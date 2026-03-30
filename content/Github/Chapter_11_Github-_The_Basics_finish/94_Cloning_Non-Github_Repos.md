# Cloning Non-Github Repos — git clone 的進階應用

## 📝 課程概述

本節延續 `git clone` 的說明，強調一個關鍵觀念：**`git clone` 是 Git 的原生命令，不是 GitHub 專屬工具**。它可以從任何代管 Git 倉庫的服務下載 Repo，包括 GitHub、GitLab、Bitbucket，甚至是自己架設的 Git 伺服器。理解這一點，能讓你靈活應對不同的開發環境。

## 核心觀念與實作解析

### `git clone` 的完整行為

當你執行 `git clone <url>` 時，Git 會：

1. **根據 URL 連接到代管伺服器**
2. **下載所有檔案與完整 Git 歷史**（所有 Commit、Branch 等）
3. **自動初始化一個新的 Git Repo**（`.git` 目錄）
4. **自動設定 Remote**：將 `origin` 指向你 Clone 的那個 URL
5. **自動切換到預設分支**（通常是 `master` 或 `main`）

### 公開 Repo 的 Clone 權限

這是初學者常見的疑惑：

> **Q：我 Clone 別人的公開 Repo，需要獲得許可嗎？**
>
> **A：不需要。** 只要能在 GitHub 上看到某個 Repo，幾乎任何人都可以 Clone 它。

但請注意：
- **Clone**：任何人都可以（如果是公開 Repo）
- **Push（提交變更）**：**不行**。未經授權無法直接 Push 到他人的 Repo
- 想貢獻他人的 Repo，需要透過 Pull Request 流程（會在後續章節介紹）

### `git clone` 不等於 GitHub

`git clone` 是 Git 的標準功能，GitHub 只是最流行的代管平台之一。其他選項包括：

- **GitLab**：類似 GitHub，有免費的私人倉庫
- **Bitbucket**：Atlassian 公司的產品
- **自架 Git 伺服器**：有些公司會自己架設 Git 服務

Git 的官方文檔中，關於 `git clone` 的說明**完全沒有提及 GitHub**，這是因為它是通用的指令。

### 實作：Clone GitLab Repo

```bash
git clone https://gitlab.com/username/project.git
```

流程與 Clone GitHub Repo 完全相同：
- Git 會下載完整 Repo 到本機
- 自動設定 Remote `origin` 指向 GitLab URL
- 自動初始化 Git Repo

### 實作：Clone GitHub Repo

```bash
git clone https://github.com/user/repo-name.git
```

講師以 Clone 熱門遊戲 2048 的 Repo 為例：
1. 從 GitHub 複製 Clone URL
2. 在終端機執行 `git clone <url>`
3. Git 會下載完整 Repo（包含 180 個 Commit 的歷史）
4. 你可以打開 `index.html` 直接在瀏覽器中玩遊戲
5. 可以任意修改本地程式碼，進行實驗

### `--depth` 參數：只 Clone 最近的部分歷史

如果 Repo 歷史很長，你只想取得最近的一些 Commit：

```bash
git clone --depth 1 https://github.com/user/large-repo.git
```

這稱為 **Shallow Clone**，只下載最近一個 Commit 的內容，省時省空間。但要注意：如果未來需要完整的 Commit 歷史，需要另外執行 `git fetch --unshallow`。

## 💡 重點摘要

- `git clone` 是 Git 的標準指令，**適用於任何代管 Git 倉庫的服務**
- 公開 Repo 任何人都可以 Clone，但未經授權不能直接 Push
- `git clone` 會自動設定 Remote `origin`，並下載完整的 Repo 歷史
- 你可以 Clone 任何你在 GitHub/GitLab/Bitbucket 等平台上能看到的 Repo
- `--depth` 可以做 Shallow Clone，只下載最近部分的歷史

## 關鍵字

git clone, remote, origin, SSH URL, HTTPS URL, shallow clone, git fetch, GitLab, Bitbucket, public repository, clone permissions
