# Demonstrating Git Fetch — Fetch 實戰演示

## 📝 課程概述

本節透過完整的實作演示，展示 `git fetch` 在真實協作場景中的運作方式。我們會在 GitHub 上直接編輯檔案（本模擬隊友的 Push），然後在本地執行 Fetch，觀察 Remote Tracking Branch 如何更新，以及如何使用 `git status` 的提示來了解本地與遠端的差距。

## 核心觀念與實作解析

### 實驗設計

**模擬情境**：你 Clone 了一個 Repo，而隊友（或者你在 GitHub 上直接編輯）在 GitHub 上新增了 Commit。

**GitHub 上的操作**：
- 在 `food` 分支新增 `apple.txt`
- 在 `movies` 分支新增 `tinkerbell.txt`

**本地觀察**：
- 剛 Clone 時：`git status` 顯示「up to date with origin/food」
- 但 GitHub 上已有新 Commit，本地還不知道

### 第一步：Fetch 之前

```bash
git status
# On branch movies
# Your branch is up to date with 'origin/movies'.
```

這句話**誤導性很強**！它說「你的分支已與 `origin/movies` 同步」，但實際上 GitHub 上有新的 Commit `origin/movies` 還不知道。這是因為 **Remote Tracking Branch 還沒有更新**。

> **為什麼 Git 不會自動更新？** 因為你的本地 Repo 不會主動去問 GitHub「有新 Commit 嗎？」——你必須明確執行 `git fetch`。

### 第二步：執行 Fetch

```bash
git fetch origin movies
# 或一次取回所有分支
git fetch origin
```

Fetch 完成後，`git status` 的訊息立即改變：

```
On branch movies
Your branch is behind 'origin/movies' by 1 commit.
```

現在 Git 告訴你：「本地 `movies` 落後 GitHub 上的 `origin/movies` 1 個 Commit。」這就是 Remote Tracking Branch 更新後帶來的準確資訊。

### 第三步：預覽遠端的變更

既然知道了差距，你可以預覽遠端到底有什麼：

```bash
git checkout origin/movies    # Detached HEAD，查看 GitHub 上的最新狀態
```

執行後，你會看到 `tinkerbell.txt` 檔案——這是 GitHub 上新增的內容，但你本地還沒有。預覽完後，隨時可以：

```bash
git switch movies    # 回到本地 movies 分支，繼續原本的工作
```

### Fetch 也能發現新分支

如果在 GitHub 上建立了一個新分支（如 `morefood`），本地一開始完全不知道：

```bash
git branch -r    # Clone 之後，還沒有 morefood
git fetch        # Fetch 更新所有 Remote Tracking Branch
git branch -r    # 現在有了：origin/morefood
```

### Fetch 只下載，不合併

這是 Fetch 與 Pull 最核心的差異：

- Fetch 後，`origin/main` 更新到 GitHub 最新狀態
- 但你的本地 `main` 分支**絲毫未動**
- 你可以自由決定什麼時候、如何合併這些變更

> **為什麼要這麼麻煩？** 因為合併是有風險的（可能衝突），Fetch 讓你**先看再做**，而不是被迫直接合併。

## 💡 重點摘要

- `git status` 說「up to date」不代表 GitHub 上沒有新 Commit——必須先 Fetch 才能確認
- Fetch 更新 Remote Tracking Branch 後，`git status` 會準確顯示「ahead」（領先）或「behind」（落後）
- Fetch 完成後，可以用 `git checkout origin/<branch>` 預覽遠端的最新狀態
- Fetch 會讓本地 Repo 知道 GitHub 上所有新建立的分支
- Fetch 不會合併任何內容，**完全安全**，隨時可以做

## 關鍵字

git fetch, git status, origin/movies, behind, ahead, git checkout origin/main, Detached HEAD, new branch, remote tracking branch, git fetch origin
