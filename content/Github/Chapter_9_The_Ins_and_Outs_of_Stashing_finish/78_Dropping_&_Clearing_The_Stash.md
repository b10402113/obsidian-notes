# Dropping & Clearing the Stash — Stash 的清理機制

## 📝 課程概述

本節介紹如何管理 Git Stash 的生命週期——不只是在需要的時候存入和取回，還包括如何**主動刪除**不再需要的 Stash 項目，以及如何**一次清空**整個 Stash Stack。這些是維護乾淨工作環境的必要操作，尤其當你頻繁使用 Stash 時。

## 核心觀念與實作解析

### 為什麼需要「刪除」Stash？

`git stash pop` 已經會自動刪除最近一次的 Stash 項目。但如果你使用的是 `git stash apply`（或 `apply` 了特定項目），Stash 記錄會**一直留在 Stack 中**，佔用空間。隨著時間推移，你的 Stash Stack 可能累積了一堆不再需要的項目，這時就需要手動清理。

### `git stash drop`：刪除指定的 Stash 項目

```bash
git stash drop stash@{n}
```

**語法**：`stash@{n}` 中的 `n` 是 `git stash list` 中顯示的編號

**使用時機**：
- 你用 `git stash apply` 取回了某個 Stash，但不再需要它
- 你 Stash 了一些實驗性變更，後來放棄了這個方向

**實作過程**：
```bash
git stash list
# stash@{0}: WIP on rainbow: abc123 (yellow)
# stash@{1}: WIP on rainbow: def456 (orange)
# stash@{2}: WIP on rainbow: ghi789 (red)

git stash drop stash@{0}
# 輸出：Dropped stash@{0} (abc123...)

git stash list
# stash@{0}: WIP on rainbow: def456 (orange)
# stash@{1}: WIP on rainbow: ghi789 (red)
```

> **注意**：`drop` 是危險操作——刪除後無法復原。與 `pop` 不同，`drop` 不會同時套用變更，只是單純移除記錄。如果你還沒套用變更就直接 `drop`，這些變更會**永久丟失**。

### `git stash clear`：一次清空整個 Stack

```bash
git stash clear
```

這個指令會**一口氣刪除 Stack 中所有 Stash 項目**，等於是把 Stash 完全清空。

**使用時機**：
- 你完成了一個大型功能的開發，確認所有 Stash 都不再需要
- 誤用了 `git stash`（意外存了一些不需要的變更）
- 要開始乾淨的新工作階段，不想讓舊的 Stash 造成混淆

**執行後**：
```bash
git stash clear

git stash list
# （無輸出，完全空白）
```

> **警告**：`git stash clear` 是破壞性極強的操作。一旦清空，所有 Stash 記錄都會消失，如果其中還有未取回的重要變更，將無法透過 Git 指令恢復。執行前請先確認 `git stash list` 的內容。

### Stash 的生命週期：何時該 drop？何時該 clear？

| 情境 | 建議指令 |
|------|----------|
| Stash 取回後，不再需要這個記錄 | `git stash drop stash@{n}`（或直接用 `pop`） |
| Stack 累積了多個廢棄 Stash | `git stash clear` |
| 不確定 Stash 裡還有沒有重要的未取回變更 | 先用 `git stash list` 確認，不要直接 `clear` |
| 想完整清空並重新開始 | `git stash clear` |

### `git stash drop` vs `git stash pop`：何時該手動刪除？

| | `git stash pop` | `git stash drop` |
|---|---|---|
| 是否套用變更 | 是 | **否** |
| 是否刪除記錄 | 是（自動） | 是 |
| 使用時機 | 取回並結束時用 | 已用 `apply` 套用後，手動清理時用 |

### 練習實作：「老闆走過來的日記」情境

講師設計了一個幽默但極具教學價值的練習，完整涵蓋了 Stash 的各個環節：

**情境**：你在 `the-truth` Branch 寫了抱怨老闆的內容，突然老闆走過來了⋯⋯

**步驟演練**：
1. 在 `master` 建立 `diary.txt`，寫入「I love my boss」→ `git add` + `git commit`
2. 建立 `the-truth` Branch，刪除內容改為「I hate my boss」（×5）
3. 老闆來了！→ `git stash`（立即把真實內容藏起來）
4. 老闆走後，在 `master` 追加更多「I love my boss」→ `git add` + `git commit`
5. 回 `the-truth` Branch → `git stash pop`（取回「I hate my boss」的內容）
6. `git add` + `git commit -m "add the truth"`

這個練習完美模擬了現實中的緊急情境，展示了 Stash 在「需要立即隱藏工作進度」時的實際用途。

## 💡 重點摘要

- `git stash drop stash@{n}`：刪除指定的 Stash 項目，**不會套用變更**
- `git stash clear`：一次清空整個 Stash Stack，**不可逆**
- `pop` 是最推薦的取回方式（取回後自動刪除）；`drop` 只在你單獨用 `apply` 後才需要
- 在執行 `drop` 或 `clear` 前，**強烈建議先執行 `git stash list` 確認內容**
- `git stash` 的日常使用中，90% 以上的情況只需要 `git stash` 和 `git stash pop` 這兩個指令

## 關鍵字

git stash drop, git stash clear, stash@{n}, git stash list, stash stack, WIP, git stash apply, git stash pop
