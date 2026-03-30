# Fixup、Squash 與 Drop：合併與刪除 Commit 的藝術

## 📝 課程概述

本單元繼續介紹 Interactive Rebase 的三個強大指令：`fixup`、`squash` 和 `drop`。`fixup` / `squash` 用於將多個相關 commit 合併為一個，保持歷史整潔；`drop` 則用於完全移除一個不想要的 commit。我們會看到這些操作的結果是 commit 記錄大幅精簡，但**程式碼變更本身完全保留**。

## 核心觀念與實作解析

### 為什麼需要合併 Commit？

實際開發中，經常會遇到這樣的 commit 序列：

```
pick abc1234 add top navbar
pick def5678 fix another navbar typo    ← 同一個功能的小修正
pick ghi9012 fix navbar typos           ← 又一個 typo 修正
```

這三個 commit 記錄了「建立 navbar」這一個完整的開發行為，但散布在三個 commit 中，不僅閱讀困難，也讓歷史顯得瑣碎。合併它們，歷史就變成：

```
pick abc1234 add top navbar             ← 包含了所有 typo 修正的內容
```

### `fixup` vs `squash`：保留一個訊息 vs 兩個訊息都留

這兩個指令都會將多個 commit 合併成一個，**但處理 commit message 的方式不同**：

| 指令 | 行為 |
|------|------|
| `squash` | 將多個 commit 合併，Git 會開啟編輯器讓你融合所有 commit message |
| `fixup` | 同樣合併 commit，但**自動拋棄被合併 commit 的訊息**，只保留目標 commit 的訊息 |

**講師推薦使用 `fixup`**：當你想要合併時，通常只需要一個乾淨的 commit message，不需要保留那些瑣碎的「fix typo」之類的中間記錄。

### 實作：`fixup` 合併 Bootstrap 兩步 commit

**原始 commits：**

```
pick abc1234 add bootstrap
pick def5678 whoops forgot to add bootstrap JavaScript script
```

目標：把第二個 commit 的**內容**（加入 Bootstrap JS 的那一行 script）合併到第一個 commit，但**不留**第二個 commit 的訊息。

**操作方式：**

在編輯器中將第二個 `pick` 改為 `fixup`（或簡寫 `f`）：

```
pick abc1234 add bootstrap
fixup def5678 whoops forgot to add bootstrap JavaScript script
```

存檔後的結果：

- 第二個 commit 的**程式碼變更**被合併進第一個 commit
- 第二個 commit 的**訊息直接消失**
- 歷史中只剩一個 `add bootstrap` commit

> 可以用 `git log --oneline` 驗證：原本的「forgot to add bootstrap js script」訊息已經不見，但打開專案看，Bootstrap JS 的 script 標籤仍然存在於 HTML 中。

### 實作：`fixup` 合併多個 typo 修正

**原始 commits：**

```
pick abc1234 add top navbar
pick def5678 fix another navbar typo
pick ghi9012 fix navbar typos
```

**操作方式：**

```
pick abc1234 add top navbar
fixup def5678 fix another navbar typo
fixup ghi9012 fix navbar typos
```

存檔後的結果：只剩一個 `add top navbar` commit，但三個 commit 的**所有程式碼變更都還在**——包括那些 typo 的修正（`Navbarr` → `Navbar`，`Linkk` → `Link`）。

> **關鍵理解**：`fixup` 從不刪除程式碼變更，它只改變 commit 的結構。程式碼內容完整保留，只是被重新分配到一個 commit 之下。

### `drop`：完全移除一個 Commit

適用情境：某個 commit 是你後悔加入的（例如：不小心 commit 了錯誤的內容、測試用的假資料、或者像課程中那樣「my cat made this commit」的趣味 commit）。

**操作方式：**

```bash
git rebase -i HEAD~2   # 只往前數 2 個 commit，因為只需要刪掉最近一個
```

在編輯器中，將該 commit 的 `pick` 改為 `drop`：

```
pick abc1234 add top navbar
drop def5678 my cat made this commit
```

存檔後：
- 該 commit **連同它的程式碼變更**一起從歷史中消失
- `git log --oneline` 不再看到這個 commit 的訊息
- 打開對應的檔案，該 commit 加入的內容也沒了（因為是 `drop` 而非 `fixup`）

### 合併操作後的 Hash 連鎖反應

如同 `reword`，任何 `fixup`、`squash`、`drop` 都改變了 commit 的結構，因此：

- 被修改的 commit 本身會獲得新 hash
- 所有在它之後的 commit，因為 parent hash 改變，全部都會變成新的 commit

驗證方式很簡單：比較 `git log --oneline` 操作前後的 commit hash，你會看到所有 commit 都換了一批新的 ID。

### 實務建議：在即將分享分支之前執行

這個 workflow 的最佳時機是：

1. 功能開發完成，所有 commit 都在本地
2. 準備 `git push` 或開 PR 之前
3. 執行一輪 `git rebase -i`，把所有瑣碎的 commit 整合完
4. 確保歷史敘述清晰、一致，符合團隊規範

一旦 push 之後，**不要再對同一段歷史執行 interactive rebase**。

## 💡 重點摘要

- `fixup` 將 commit 內容合併進前一個 commit，並**自動拋棄被合併 commit 的訊息**；`squash` 則會開啟編輯器讓你融合訊息
- `fixup` / `squash` 只改變 commit 的**結構**，所有程式碼變更完整保留，只是被分配到新的 commit 之下
- `drop` 完全移除一個 commit，包括它的程式碼變更——適用於刪除不想要的 commit，而非修復
- 合併或刪除 commit 後，該 commit 以及所有後續 commit 都會獲得新的 hash
- **所有這些操作都應在程式碼尚未與他人分享之前執行**，這是 local cleanup 的黃金時機

## 關鍵字

`fixup`, `squash`, `drop`, `git rebase -i`, commit merging, commit squashing, commit hash, parent hash, rewrite history, feature branch cleanup, local workflow
