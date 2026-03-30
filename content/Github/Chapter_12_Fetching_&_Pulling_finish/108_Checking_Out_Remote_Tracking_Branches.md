# Checking Out Remote Tracking Branches — 理解 Remote Tracking Branch

## 📝 課程概述

本節深入說明 Clone 背後的運作機制，尤其是當你 Clone 一個 Repo 時，Git 實際上為你做了什麼。我們會詳細介紹 **Remote Tracking Branch**（遠端追蹤分支）的概念，以及 `git branch -r` 這個查看所有 Remote Tracking Branch 的指令。理解這個概念，是理解 Fetch 和 Pull 如何運作的基礎。

## 核心觀念與實作解析

### Clone 時 Git 做了什麼？

當你執行 `git clone <url>` 時，Git 會：
1. 下載所有 Commit 歷史與檔案
2. 建立一個 Git Repo
3. **自動設定 Remote** `origin` 指向 Clone 來源的 URL
4. **自動設定 Remote Tracking Branch**（如 `origin/main`）

### Remote Tracking Branch 是什麼？

Remote Tracking Branch 是 Git 用來**記憶遠端 Repo 狀態**的指標。Clone 完成後，你的本地有兩種分支：

```
main          → 本地分支指標（你可以自由移動它）
origin/main   → Remote Tracking Branch（Git 用它記住遠端的狀態）
```

> **比喻**：把 `main` 看成「你手裡的書籤」，把 `origin/main` 看成「圖書館管理員的書籤」。你可以移動自己的書籤，但圖書館管理員的書籤只有在他們告訴你更新時才會改變。

### 為什麼需要 Remote Tracking Branch？

Remote Tracking Branch 的存在讓你可以在不影響本地工作的情況下，**預覽遠端有哪些變更**：

- 當你在 `main` 分支上做 Commit 時，`main` 向前移動
- 但 `origin/main` **不動**，因為你還沒有和 GitHub 通信
- `git status` 會顯示「Your branch is ahead of 'origin/main' by N commit(s)」
- 這句話的意思是：「你領先了 GitHub N 個 Commit，但 GitHub 上可能有更多你還不知道的 Commit」

### `git branch -r`：查看所有 Remote Tracking Branch

```bash
git branch -r
# 輸出：
#   origin/HEAD -> origin/main
#   origin/fantasy
#   origin/food
#   origin/main
#   origin/movies
```

Clone 完成時，Git 會自動為**每一個遠端分支**建立對應的 Remote Tracking Branch。即使你 Clone 後只能看到 `main`/`master` 一個本地分支，`git branch -r` 可以看到所有 GitHub 上存在的分支。

### 為什麼 Clone 後本地只有一個分支？

Clone 時，Git **不會把所有遠端分支都變成你的本地分支**。它只會：
1. 把預設分支（`main`/`master`）Checkout 出來，成為你的本地分支
2. 其餘分支建立 Remote Tracking Branch（`origin/xxx`），讓你知道它們存在，但不自動建立本地分支

### 離開 Detached HEAD 的方法

如果你 Checkout 了 `origin/main`，會進入 Detached HEAD 狀態（因為 Remote Tracking Branch 不是真正的本地分支）。離開方式：

```bash
git switch main      # 回到本地 main 分支
```

### `git status` 中的遠端狀態提示

Clone 後執行 `git status`，Git 會告訴你本地分支與遠端的差距：

```
On branch main
Your branch is up to date with 'origin/main'.
```

當你做了新 Commit 但還沒 Push 時：

```
On branch main
Your branch is ahead of 'origin/main' by 2 commits.
```

這句話的含義是：「你的 `main` 領先於 GitHub 上的 `main` 2 個 Commit，但 GitHub 上可能有別人的新 Commit 你還沒取回。」

## 💡 重點摘要

- Clone 時，Git 除了建立本地分支外，還會自動設定 Remote Tracking Branch（如 `origin/main`）
- Remote Tracking Branch 是 Git 用來**記憶遠端 Repo 狀態**的指標，不會隨本地操作自動移動
- `git branch -r` 可以查看所有 Remote Tracking Branch
- `git status` 會根據 Remote Tracking Branch 告訴你本地分支領先或落後 GitHub 多少 Commit
- `origin/main` 和本地 `main` 是**兩個不同的指標**，它們一開始指向同一個 Commit，後來可能會分道揚鑣

## 關鍵字

git branch -r, remote tracking branch, origin/main, origin/master, origin/fantasy, git clone, git status, detached HEAD, git checkout origin/main, default branch
