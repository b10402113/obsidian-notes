# What Really Matters in This Section — Undoing Changes & Time Traveling

## 📝 課程概述

本節是整個課程中指令數量最多的章節之一，涉及多個名稱相似但功能不同的 Git 指令——`git checkout`、`git restore`、`git reset`、`git revert`，以及 Detached HEAD 的概念。它們共同的核心用途是：**幫助你在不同情境下「回到過去」或「撤銷變更」**。本節的重點在於理解每個指令的適用場景，特別是 `reset` 和 `revert` 在協作情境下的取捨。

## 核心觀念與實作解析

### 本節指令一覽

| 指令 | 主要用途 | 重要性 |
|------|----------|--------|
| `git checkout <commit-hash>` | 檢視歷史 Commit（進入 Detached HEAD） | Important |
| `git checkout HEAD -- <file>` | 將檔案還原至上一次 Commit 的狀態 | Important |
| `git restore <file>` | 將檔案還原至 HEAD（新語法，等同於 checkout 的部分功能）| Important |
| `git restore --staged <file>` | 取消 Staging（將檔案移出 Staging Area）| Important |
| `git reset <commit-hash>` | 刪除指定範圍內的 Commits，**保留** Working Directory 的變更 | Important |
| `git reset --hard <commit-hash>` | 刪除指定範圍內的 Commits，**連同 Working Directory 的變更一起清除** | Important |
| `git revert <commit-hash>` | 透過**新增一個新 Commit** 來撤銷某個舊 Commit 的變更 | Important |

### 為什麼一次介紹這麼多指令？

這些指令名稱相似（「都有『回』的意思」），但行為差異極大。Git 社群在這方面累積了多年的痛點，才催生了 `git switch` 和 `git restore` 這兩個新的專用指令，來分擔 `git checkout` 過於龐大的職責。

講師的建議是：不需要把每個指令的每個選項都背起來，但必須理解**何時該用哪一個**，以及**它們對 Commit 歷史和 Working Directory 的影響**。

### Detached HEAD：進入歷史的門票

當你執行 `git checkout <commit-hash>` 而非 `git checkout <branch>` 時，Git 會進入所謂的 **Detached HEAD** 狀態。這不是錯誤，是一個特殊的「觀看模式」：

- **正常狀態**：HEAD 指向一個 Branch Reference（分支引用），Branch Reference 再指向某個 Commit
- **Detached HEAD**：HEAD 直接指向一個 Commit，不再綁定任何分支

這種狀態下你可以自由地「觀看」歷史，但**不應該直接在上面做 Commit**（除非你打算之後建立新分支）。離開的方式：切換回任意分支即可。

### Reset vs. Revert：協作場景下的關鍵抉擇

這是本章最重要的觀念對比：

- **`git reset`**：直接刪除 Commit，Branch Pointer 向後移動——**改寫歷史**
- **`git revert`**：新增一個「反轉變更」的 Commit，保留原始 Commit 的歷史——**安全地添加新 Commit**

> **核心問題**：如果你已經把 Commit Push 到 Remote，並且同事已經基於這些 Commit 做了後續開發，這時用 `reset` 會造成嚴重的歷史衝突。**在協作環境中，`revert` 是更安全的選擇。**

## 💡 重點摘要

- 本節所有指令都是為了「回到過去」或「撤銷變更」，但它們的作用範圍和風險差異極大
- Detached HEAD 不是錯誤，是一種「純閱讀歷史」的狀態，離開時切回分支即可
- `git reset` **不改變 Working Directory**；`git reset --hard` **會清除 Working Directory 的變更**
- `git revert` 透過新增一個 Commit 來撤銷，安全適用於已分享的歷史
- `git restore` 是 Git 新增的專用指令，用於還原檔案和取消 Staging，語義比 `git checkout` 更清晰

## 關鍵字

git checkout, git restore, git reset, git revert, detached HEAD, git reset --hard, git restore --staged, history rewriting, HEAD~n, commit hash
