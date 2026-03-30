# Git Stash Apply — 跨 Branch 套用 Stash 的進階技巧

## 📝 課程概述

本節介紹 `git stash apply` 與 `git stash list` 這兩個進階工具，並深入探討「多個 Stash」的 Stack 管理機制。`apply` 和 `pop` 的核心差異在於：`apply` 會**保留** Stash 記錄（可用於跨多個 Branch 重複套用），而 `pop` 會**刪除** Stash 記錄。這兩個指令各自適用於不同的情境。

## 核心觀念與實作解析

### `git stash apply` 與 `git stash pop` 的根本差異

| 指令 | 從 Stack 取回？ | 刪除 Stack 項目？ | 適用場景 |
|------|----------------|-------------------|----------|
| `git stash pop` | 是 | **是**（自動刪除） | 大多數情境，取回後不再需要這個 Stash |
| `git stash apply` | 是 | **否**（保留在 Stack） | 需要在多個 Branch 重複套用同一組變更 |

> **核心概念**：`pop` 是「用完即丟」的衛生紙，`apply` 是「留著備用」的工具箱。

### 實作示範：何時該用 `apply` 而非 `pop`？

講師設計了一個生動的情境：

**情境**：你在 `goodbye` Branch 上有一些紫色樣式變更，現在你想：
1. 先把這些變更 stash 起來
2. 切換到 `master` Branch，**套用一次**這些變更（可能是測試）
3. 回到 `goodbye` Branch，再**套用一次**同樣的變更

**如果是 `git stash pop`**：
- 第一次套用後，Stash Stack 中的該項目就被刪除了
- 第二次套用時，已經沒有 Stash 可以取回了

**如果是 `git stash apply`**：
- 第一次套用後，Stash Stack 中的項目**仍然存在**
- 回到 `goodbye` Branch 後，可以再次執行 `git stash apply`，變更會再次被套用

```bash
# 存入 Stash
git stash

# 切換到 master 並套用
git switch master
git stash apply    # Stash 仍保留在 Stack 中

# 處理完，回到原 Branch 再次套用
git switch goodbye
git stash apply    # 再次套用，Stack 中該項目仍然存在
```

### Stash Apply 的衝突處理

`git stash apply` 和 `pop` 在合併變更時，都可能遇到衝突。這時的處理方式與 Merge Conflict 完全相同——Git 會標記衝突檔案，你需要手動選擇保留哪個版本：

```bash
git stash apply
# 輸出：Auto-merging index.html
# 輸出：CONFLICT (content): Merge conflict in index.html
```

遇到衝突時，你可以：
1. 開啟衝突檔案，手動編輯並解決衝突
2. 執行 `git add <file>` 標記為已解決
3. 如需要，執行 `git commit` 完成合併

### 管理多個 Stash：Stack 的概念

Git 的 Stash 不是一個「單一容器」，而是一個 **Stack（棧）**——先進後出（Last In, First Out）。

每次執行 `git stash`，新的變更會被**壓入** Stack 的頂端：
- 執行第 1 次 `git stash`：存入 `stash@{0}`
- 執行第 2 次 `git stash`：存入 `stash@{0}`，原本的變成 `stash@{1}`
- 執行第 3 次 `git stash`：存入 `stash@{0}`，Stack 順序變成 0 → 1 → 2

### `git stash list`：查看所有 Stash 項目

```bash
git stash list
# 輸出：
# stash@{0}: WIP on rainbow: abc123 remove background color
# stash@{1}: WIP on rainbow: def456 remove background color
# stash@{2}: WIP on rainbow: ghi789 remove background color
```

- `stash@{0}`：**最近一次**存入的 Stash（`pop` 預設取回的目標）
- `stash@{1}`：倒數第二次存入的 Stash
- `stash@{2}`：更早的 Stash

### 指定套用特定 Stash：`stash@{n}` 語法

當你需要取回的不是最近一次 Stash，而是更早的某個 Stash 時：

```bash
# 只套用 stash@{2}（紅色背景的 Stash），不刪除它
git stash apply stash@{2}

# 套用並刪除
git stash pop stash@{2}
```

### 情境練習：連續 Stash 三個版本的背景顏色

講師演示了一個相當極端的例子：

1. 將 `app.css` 的背景設為紅色 → `git stash`（存入 `stash@{0}`）
2. 將背景改為橙色 → `git stash`（存入 `stash@{0}`，紅色變成 `stash@{1}`）
3. 將背景改為黃色 → `git stash`（存入 `stash@{0}`，橙色變成 `stash@{1}`，紅色變成 `stash@{2}`）

執行 `git stash list` 可看到三個 Stash 項目：
```
stash@{0}: WIP on rainbow: ... (yellow)
stash@{1}: WIP on rainbow: ... (orange)
stash@{2}: WIP on rainbow: ... (red)
```

> **實用提醒**：這種連續 Stash 的情況在日常開發中並不常見。更常見的仍然是「一次 Stash，一次取回」的簡單流程。但如果你的工作流程確實需要在多個分支之間來回套用相同的實驗性變更，這個機制會非常有用。

## 💡 重點摘要

- `git stash apply` 取回變更但**保留** Stack 項目；`pop` 取回後**自動刪除**
- `apply` 適合需要在**多個 Branch 重複套用**同一組變更的場景
- `git stash list` 顯示所有 Stash，編號由新到舊：`stash@{0}` 是最近一次
- 使用 `git stash apply stash@{n}` 語法可以精確取回特定 Stash
- `apply` 和 `pop` 都可能遇到 Merge Conflict，處理方式與一般 Merge 相同
- 大多數開發者 90% 以上的時間只需要 `git stash` + `git stash pop`

## 關鍵字

git stash apply, git stash pop, stash@{n}, git stash list, stash stack, WIP, merge conflict, multiple stashes, git stash save
