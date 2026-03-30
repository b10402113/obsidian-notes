# Forking——在沒有存取權限的情況下複製專案

## 📝 課程概述

本單元介紹 **Forking**——一個由 GitHub（而非 Git）提供的功能，讓你可以複製任何公開的 repository 到自己的 GitHub 帳戶下。Fork 是大型開源專案協作的起點：你不會獲得原專案的 push 權限，但你會得到一個完整的、可自由操作的个人 fork——在這裡你可以任意實驗、commit、push，然後透過 PR 將你的建議提交給原專案。

## 核心觀念與實作解析

### Fork 不是 Git 的功能

**Fork 是 GitHub（以及 Bitbucket、GitLab 等）提供的平台功能，不是 Git 的原生命令。**

當你在 GitHub 上點擊 Fork 按鈕時，GitHub 會：
1. 在你的帳戶下建立一個新的 repository
2. 把原專案的**所有資料與歷史**完整複製過去
3. 設定 fork 關係（GitHub 知道這個 fork 是從哪裡 fork 出來的）

### Fork 解決了什麼問題？

想象你要對 React 專案貢獻一個新功能：

- React 的 repository 有**上百位 maintainer**，數千位 contributor
- 讓每一個 contributor 都成為 collaborator、擁有 push 權限 → **絕對不可能**
- 但開源專案需要外部貢獻 → **必須有一種方式讓陌生人參與**

**Fork 就是這個問題的答案**：每個人都可以 fork，自己做實驗，出錢、出力之後透過 PR 提交。

### Fork 的實作過程

**步驟 1：在 GitHub 上 Fork 專案**

在你想貢獻的 repository 頁面點擊 **Fork** 按鈕。GitHub 會問你：「要把這個 fork 到哪個帳戶？」

選擇你的帳戶後，GitHub 會建立：

```
https://github.com/YOUR_USERNAME/REPO_NAME
```

> 原本的專案（原專案稱為 **upstream**）不會被任何影響，你的 fork 是完全獨立的副本。

**步驟 2：Clone 你的 Fork**

```bash
$ git clone https://github.com/YOUR_USERNAME/REPO_NAME.git
$ cd REPO_NAME
```

**重要差異**：當你 clone fork 時，預設的 remote `origin` 會自動指向**你的 fork**，而非原專案。

```bash
$ git remote -v
origin    https://github.com/YOUR_USERNAME/REPO_NAME.git (fetch)
origin    https://github.com/YOUR_USERNAME/REPO_NAME.git (push)
```

**步驟 3：在 Fork 上自由作業**

```bash
# 在 fork 上開發、commit，完全不需要任何人的許可
$ git add .
$ git commit -m "improve something"
$ git push origin main    # push 到你的 fork，origin 正是你的 fork
```

> **你在 fork 上的所有操作都與原專案無關**，你可以任意刪除 branch、破壞程式碼、修改歷史——原專案的 maintainer 完全不會受到影響。

### 設定 Upstream Remote

你的 fork 是**靜止在 fork 時的原專案狀態**，原專案未來有新的 commit，你的 fork 不會自動同步。你需要手動設定一個 `upstream` remote：

```bash
# 加入原專案（原 upstream）作為第二個 remote
$ git remote add upstream https://github.com/ORIGINAL_OWNER/REPO_NAME.git

$ git remote -v
origin      https://github.com/YOUR_USERNAME/REPO_NAME.git (fetch/push)
upstream    https://github.com/ORIGINAL_OWNER/REPO_NAME.git (fetch only)
```

> **習慣命名**：`origin` = 你的 fork，`upstream` = 原專案。這是社群約定俗成的命名方式。

### 從 Upstream 取得最新變更

當原專案有新 commit 時，你的 fork 會落後。透過 `upstream` remote 來同步：

```bash
$ git fetch upstream
$ git switch main
$ git merge upstream/main   # 把 upstream 的最新變更 merge 到你的 local main
$ git push origin main      # 再把你的 main push 到你的 fork
```

這個流程確保你的 fork 包含了原專案的最新變更，讓你的 PR 基於最新的程式碼。

### Fork 的限制

- **你無法直接 push 到原專案**（這是 fork 的設計目的）
- **Fork 是「快照」，不會自動同步原專案的更新**——需要手動 fetch upstream 來同步
- **開源專案通常有 Contribution Guidelines**：每個 PR 可能有嚴格的格式要求、測試要求，fork 後請先詳讀

## 💡 重點摘要

- **Fork = GitHub 上的 repository 複製**，它創造一個屬於你自己的完整副本，讓你可以在沒有 push 權限的情況下自由操作。
- **Fork 的最大價值**：讓任何人都可以對開源專案做出貢獻，而不需要被給予 collaborator 權限。
- **Clone fork 之後，origin 自動指向你的 fork**——這就是你能夠 `git push` 的原因。
- **設定 upstream remote 是維持 fork 同步的關鍵**：原專案有新變更時，你需要 fetch upstream 並 merge 到你的 local main，保持 fork 與原專案的差距不會太大。
- **Fork 之後，你可以在自己的 fork 裡任意破壞**：commit history 可以重寫、branch 可以刪除，這些都不會影響原專案。

## 關鍵字

Fork, GitHub Fork, Upstream Remote, Origin Remote, git remote add, git fetch upstream, Pull Request from Fork, Open Source Contribution, GitHub Repository, Contributor, Maintainer, Snapshot, Sync Fork
