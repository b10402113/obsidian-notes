# Stashing Basics — Git Stash Save & Pop 實戰

## 📝 課程概述

本節是 Stash 章節的核心實作課。我們將深入理解 `git stash` 和 `git stash pop` 這兩個最常用的指令——前者把「無論是否已 staging」的未 commit 變更全部存進 Stash Stack，後者則把最近一次 Stash 取回來並從 Stack 中刪除。理解這兩個指令的配合使用，就能應付日常 90% 以上的 Stash 需求。

## 核心觀念與實作解析

### Stash 的運作機制

當你執行 `git stash` 時，Git 會做以下事情：

1. **蒐集**：收集所有「未 commit 的變更」，包括已 staging 和未 staging 的內容
2. **暫存**：把這些變更放進一個名為 Stash 的 Stack（棧）結構中
3. **回退**：將 Working Directory 還原到「與上次 Commit 完全一致」的狀態

> **關鍵理解**：Stash 並不是 Commit。Commit 會進入正式的專案歷史，Stash 只是暫時存放。Commit 記錄了完整的故事，Stash 只是「我還沒準備好要記錄下來的臨時狀態」。

### `git stash` 與 `git stash save` 的關係

```bash
git stash        # 完整寫法的簡寫
git stash save  # 完整寫法，兩者功能完全相同
```

兩者等價，講師建議用 `git stash` 即可，語法更簡潔。

### `git stash pop` 的運作邏輯

```bash
git stash pop
```

`pop` 的名稱來自 Stack 資料結構的「彈出」操作：
- 從 Stash Stack 中取出**最近一次**存入的變更
- 將這些變更**應用到目前 Branch 的 Working Directory**
- 從 Stack 中**移除**這個項目

> **為什麼用 `pop` 而非一直用 `apply`？** 因為 `pop` 會自動清理 Stack，而 `apply` 不會。如果你用 `apply` 後忘記手動刪除，Stash Stack 就會累積越來越多無效項目。

### 實作示範：完整的 Stash 工作流

以下是一次完整的 Stash 情境演練：

**Step 1：在 `goodbye` Branch 上有未 commit 的變更**
- `index.html`：標題從「Hello World」改為「Goodbye World!!」
- `app.css`：背景顏色設為 `magenta`

**Step 2：執行 `git stash`**
```bash
git stash
# 輸出：Saved working directory and index state WIP on goodbye: abc123 initial commit
```

執行後：
- `git status`：顯示 `working tree clean`，沒有任何未 commit 的變更
- `index.html`：恢復成「Hello World」（`goodbye` 版本的內容）
- `app.css`：magenta 背景消失，恢復原狀

**Step 3：切換到其他 Branch**
```bash
git switch master
# 現在可以順利切換，不會有衝突問題
```

**Step 4：處理完緊急事務後，取回 Stash**
```bash
git switch goodbye     # 回到原本的 Branch
git stash pop         # 取出最近一次 Stash 並應用
```

`stash pop` 執行後：
- `index.html` 重新顯示「Goodbye World!!」
- `app.css` 重新顯示 `magenta` 背景
- 變更的狀態與 Stash 前相同（unstaged）
- Stash Stack 中的該項目被移除

### 為什麼不直接 Commit？

這是一個很重要的觀念問題。 Commit 是專案歷史的永久記錄，應該代表「有意義的功能或修復」。如果你的變更只是「工作進行到一半，隨時可能被放棄」的情況，隨意 Commit 會讓歷史變得雜亂：

- **太多無價值的 Commit**：未完成的實驗性改動會進入歷史，造成混淆
- **不利於 Code Review**：別人看到這些半成品 Commit，會困惑你究竟完成了什麼
- **難以清理**：刪除這些 Commit 比放棄一個 Stash 要困難得多

Stash 就是為這個情境設計的——**把未完成的狀態像書籤一樣標記，之後再回來繼續**。

### `git stash` 的範圍：同時包含 Staged 和 Unstaged 變更

這是一個容易被忽略的細節：`git stash` 預設會把「所有」未 commit 的變更都存起來，包括：

- **已 staging 的變更**（已執行 `git add` 但尚未 `git commit`）
- **未 staging 的變更**（修改後從未 staging）

兩者都會一起存入 Stash，取回後兩者都恢復為 unstaged 狀態。

## 💡 重點摘要

- `git stash`：把「所有未 commit 的變更」存入 Stash Stack，並將 Working Directory 恢復乾淨
- `git stash pop`：取出最近一次 Stash 並**刪除**該 Stack 項目
- `git stash` **同時包含** staged 和 unstaged 兩種未 commit 的變更
- Stash 和 Commit 的最大區別：Stash 是臨時的、不進入歷史；Commit 是永久的、進入歷史
- `git stash pop` 是推薦的取回方式，因為它會自動清理 Stack；`git stash apply` 不會自動刪除

## 關鍵字

git stash, git stash pop, git stash save, working directory, staging area, stash stack, WIP, uncommitted changes, git status
