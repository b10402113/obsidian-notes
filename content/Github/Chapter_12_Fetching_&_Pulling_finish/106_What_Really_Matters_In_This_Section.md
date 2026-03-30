# What Really Matters in This Section — Fetching & Pulling 總覽

## 📝 課程概述

本節是 GitHub 基礎的延續，聚焦於「如何將變更從 GitHub 取回到本地」——這是多人協作的關鍵能力。`git push` 讓你把本地變更推送上去，而 Fetch 和 Pull 則讓你取得協作者在 GitHub 上推進的最新變更。本節包含三個核心觀念：**Remote Tracking Branches**（遠端追蹤分支）、**Git Fetch**（取回但不合併）和 **Git Pull**（取回並合併），每個都是 Critical 等級。

## 核心觀念與實作解析

### 為什麼需要 Fetch 和 Pull？

當你與他人共同開發一個專案時，事情不會只是你單向地 Push 到 GitHub——**其他人的變更也會被 Push 到 GitHub**。你需要一個方式把那些變更取回到你的本地機器上，這正是 Fetch 和 Pull 存在的意義。

### 本節學習地圖

| 主題 | 指令 | 重要性 |
|------|------|--------|
| Remote Tracking Branches | `origin/master`、`git branch -r` | ⭐⭐⭐ Critical |
| Git Fetch | `git fetch <remote>` | ⭐⭐⭐ Critical |
| Git Pull | `git pull <remote> <branch>` | ⭐⭐⭐ Critical |

### Remote Tracking Branches 的概念

Clone 一個 Repo 後，你的本地不只有 `main`/`master` 分支——還有一個名為 `origin/main`（或 `origin/master`）的 Remote Tracking Branch。這是一個**指標（Pointer）**，Git 用它來記住：「上一次與 GitHub 通信時，origin 的 main 分支在什麼 Commit」。

> **為什麼需要這個指標？** 因為你的本地 Repo 不會自動知道 GitHub 上發生了什麼變更。Remote Tracking Branch 就是那個「書籤」，讓你知道本地與遠端之間的差距。

### Fetch vs. Pull 的核心區別

| 指令 | 行為 | 對 Working Directory 的影響 |
|------|------|------------------------|
| `git fetch` | 下載遠端的新 Commit，更新 Remote Tracking Branch | **不影響** Working Directory（安全）|
| `git pull` | 等同於 `fetch` + `merge`，下載並合併到本地分支 | **直接合併**到 Working Directory（有衝突風險）|

> **Fetch = 先預覽再決定；Pull = 直接合併**。在你不確定的情況下，先 Fetch 總是更安全的選擇。

## 💡 重點摘要

- 多人協作時，其他人的變更會被 Push 到 GitHub，你需要 Fetch 或 Pull 來取回這些變更
- Remote Tracking Branch（如 `origin/main`）是 Git 用來記住「遠端分支上次在哪個 Commit」的指標
- `git fetch` 安全地下載遠端變更到 Remote Tracking Branch，**不影響 Working Directory**
- `git pull` = `fetch` + `merge`，會將遠端變更直接合併到本地分支，可能產生衝突
- **建議**：在 Pull 之前先 Fetch，確認遠端有哪些變更，再決定是否合併

## 關鍵字

git fetch, git pull, remote tracking branch, origin/main, origin/master, git merge, git branch -r, git push, collaboration, download changes, upstream
