# Git Fetch — 安全的預覽工具

## 📝 課程概述

本節介紹 `git fetch`——從遠端取回最新 Commit 資訊，但**不影響 Working Directory** 的指令。Fetch 是「安全預覽」的工具：在你決定合併之前，先看看 GitHub 上有哪些變更，而不會打亂你目前在本地的工作。

## 核心觀念與實作解析

### Fetch 的運作方式

```
Remote Repository (GitHub)     Local Repository (.git folder)
          │                                  │
          │    git fetch                     │
          │ ─────────────────────────────────>
          │     (下載新 Commit，更新         │
          │      Remote Tracking Branch)      │
          │                                  │
          │                                  ▼
          │                         Remote Tracking Branch
          │                              (origin/main)
          │                                  ▲
          │                                  │
          │                        Local Branch (main)
          │                        不受影響
```

**Fetch 做的事情**：
1. 與指定的 Remote（通常是 `origin`）通信
2. 詢問：「GitHub 上的這個分支有哪些我還沒有記錄的 Commit？」
3. 下載這些新 Commit
4. **更新 Remote Tracking Branch**（如 `origin/main`）
5. **完全不觸碰 Working Directory**

### Fetch 的語法

```bash
# 取回所有 Remote 的所有分支
git fetch origin

# 取回所有 Remote 的指定分支
git fetch origin main

# 取回所有 Remote 的所有分支
git fetch   # 如果只有一個 Remote（origin），可省略
```

### Fetch 之後會發生什麼？

執行 `git fetch` 後：
- `git status` 的訊息改變：可能從「ahead」變成「behind」或「have diverged」
- **可以預覽**：`git checkout origin/main`（Detached HEAD）查看 GitHub 上最新版本的內容
- **可以建立分支**：`git switch -c new-branch origin/main` 在 GitHub 的最新狀態上建立新分支

### 為什麼 Fetch 是安全的？

因為 Fetch **完全不會改變你的 Working Directory**。你本地正在修改的檔案、正在編寫的程式碼，統統不受到影響。

> **比喻**：Fetch 就像去圖書館查詢系統看一下「目前這本書在哪一頁」，但你**沒有把它借走**。你仍然在自己手邊的書上繼續工作。

### Fetch 的常見使用時機

1. **協作者剛 Push 了新內容**：你想先看看他們做了什麼，再決定是否合併
2. **不確定 GitHub 上是否有更新**：先 Fetch，再根據 `git status` 的提示決定下一步
3. **新分支被建立**：讓本地 Repo 知道 GitHub 上有哪些新分支存在
4. **Pull 之前先 Fetch**：確認即將合併的內容，減少衝突風險

### Fetch 與本地/遠端分支的關係圖解

**情境**：你本地落後 GitHub 2 個 Commit

```
Fetch 前：
  main                → Commit C (本地最新)
  origin/main         → Commit A (GitHub 上的狀態，停在這裡)

Fetch 後：
  main                → Commit C (本地最新，仍不動)
  origin/main         → Commit E (更新到 GitHub 最新狀態)
```

Fetch 後你仍然在 Commit C 上，但 `origin/main` 現在指向 Commit E。你可以在這個狀態下 `git checkout origin/main` 預覽 Commit E 的內容，或者執行 `git pull` 將這些變更合併過來。

## 💡 重點摘要

- `git fetch` 下載遠端的新 Commit，**更新 Remote Tracking Branch**，**不影響 Working Directory**
- Fetch 完成後，可以用 `git checkout origin/main`（Detached HEAD）預覽遠端最新狀態
- Fetch 是完全安全的操作，**不會打亂你本地正在進行的工作**
- `git fetch` 也會更新所有 Remote Tracking Branch，包括那些你還沒有取成本地分支的遠端分支
- **建議**：不確定遠端是否有變更時，先 Fetch，再根據 `git status` 的提示行動

## 關鍵字

git fetch, origin/main, remote tracking branch, git checkout origin/main, git status, git fetch origin, git fetch origin main, safe operation, preview remote changes
