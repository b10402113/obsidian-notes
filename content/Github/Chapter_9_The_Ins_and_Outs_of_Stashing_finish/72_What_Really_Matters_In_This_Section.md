# What Really Matters in This Section — Git Stash 總覽

## 📝 課程概述

本節介紹 Git 中一個「方便但非必要」的工具：`git stash`。當你在某個 Branch 上有些尚未完成的修改，卻需要緊急切換到另一個 Branch 處理其他事務時，`git stash` 可以讓你把這些「未 commit 的變化」暫時儲存起來，稍後再取回。這不是解決問題的唯一方式，但在許多情境下非常順手。

## 核心觀念與實作解析

### 為什麼需要 Stash？

在日常開發中，你可能會遇到這樣的場景：你在某個 Branch 上埋頭修改，但突然需要緊急切換到另一個 Branch——也許是同事請求幫忙，也許是線上環境出現了 Bug 需要立即處理。這時你有幾個選項：

1. **把目前的修改 commit 掉**：但你還沒準備好做正式 commit，因為工作只完成了一半
2. **什麼都不做，直接切換**：Git 可能會**拒絕讓你切換**（當目標 Branch 上存在衝突的檔案時），或者你的未 commit 變化會**跟著你一起帶過去**，造成污染
3. **使用 `git stash`**：把未 commit 的變化暫存起來，等處理完其他事情後再取回

### Stash 的核心時機

根據講師的說法，Stash 比較像是「便利工具」（Convenience Method），而非每天都會用到的核心指令。並非所有開發者都會頻繁使用它，但知道它的存在，能讓你在需要的時候多一個選擇。

常見使用時機：
- 同事請求你緊急協助，切換 Branch 前不想留下半成品
- 你在修改某個檔案，但發現這個 Bug 需要在另一個 Branch 才能重現
- 你正在 debug，但同時想嘗試一個乾淨的環境來驗證假設

### Git Stash 的兩種使用情境

講師用兩個情境清楚展示了 Stash 解決的問題：

**情境一：未衝突的變更可以跟著你走**
- 在 `purple` Branch 修改 `app.css`
- 切換到 `master`：Git **允許**這個操作，你的變更會「跟著你」帶到 `master` 上
- 這不見得是好事——你可能不想把這些紫色樣式帶到 `master`

**情境二：衝突時 Git 會拒絕讓你切換**
- 在 `goodbye` Branch 將 `index.html` 的標題改為「Goodbye」
- 同時在 `master` 也修改了 `index.html`（改成「Hello World!!」）
- Git 拒絕切換，並顯示：`Your local changes to the following files would be overwritten by checkout. Please commit your changes or stash them before you switch branches.`

> 這就是 `git stash` 登場的時刻——Stash 可以幫你「暫存」這些衝突中的變更，讓你順利切換 Branch，之後再把變更取回來。

### 本節學習地圖

| 指令 | 等級 | 說明 |
|------|------|------|
| `git stash` / `git stash save` | **Important** | 將未 commit 的變更存入 Stash |
| `git stash pop` | **Important** | 取回最近一次 Stash 並從 Stack 移除 |
| `git stash apply` | Nice to Have | 取回 Stash 但保留在 Stack 中 |
| `git stash drop` | Nice to Have | 手動刪除某個 Stash 項目 |
| `git stash clear` | Nice to Have | 一次清空整個 Stash |
| 多重 Stash 管理 | Nice to Have | 理解 `git stash list` 與 `stash@{n}` 語法 |

## 💡 重點摘要

- `git stash` 解決的核心問題是：**不想 commit 但又需要切換 Branch**
- Stash 不是必要工具，但掌握它能在緊急情境下讓你更從容
- 最關鍵的兩個指令是 `git stash`（存）和 `git stash pop`（取回並刪除）
- `git stash apply` 可以取回變更但**保留** Stash 記錄，適用於需要在多個 Branch 套用同一組變更的場景
- 理解「未衝突變更可以帶著走」與「衝突時 Git 會阻擋切換」兩種行為，是掌握 Stash 的前提

## 關鍵字

git stash, git stash save, git stash pop, git stash apply, working directory, staging area, uncommitted changes, branch switching, stash list, stash@{n}
