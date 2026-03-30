# Diff Exercise — 實作練習與指令總整理

## 📝 課程概述

本節是 Chapter 8 的實作練習課。我們透過一個預先準備好的 Repository（含 `1970s` 和 `current` 兩個分支，分別代表樂團在不同年代的陣容），實際操作所有 `git diff` 的變化形式。這個練習的價值在於：讓你親手體驗 `git diff`、`git diff --staged`、`git diff HEAD` 的輸出差異，並理解 Branch 比較、背後 Commit 比較等各種情境。

## 核心觀念與實作解析

### 練習環境說明

這個練習使用一個名為 `git diff exercise` 的 Repository，裡面有兩個文字檔案：`fleetwoodmac.txt` 和 `queen.txt`，分別代表 Fleetwood Mac 和 Queen 兩個樂團的成員資料。Repository 預設有兩個分支：

- **`current`**：目前（2021–2022）的現代陣容
- **`1970s`**：1970 年代的經典陣容

> **注意**：在影片中，`git branch` 一開始**不會顯示** `1970s` 分支——這是因為 `1970s` 是一個 Remote Branch。這個現象的原因會在 Chapter 12（Fetching & Pulling）中介詳細說明。目前你只需要執行 `git switch 1970s` 即可切換到該分支。

### 練習 1：比較兩個 Branch（所有檔案）

```bash
git diff 1970s..current
```

這會比較 `1970s` 分支與 `current` 分支之間**所有檔案**的差異。從輸出中你可以看到：
- Fleetwood Mac：`1970s` 沒有 Keyboardist，`current` 有 Christine McVie
- Fleetwood Mac：`current` 沒有 Lindsey Buckingham（`1970s` 有）
- Queen：`1970s` 的 Bass 是 John Deacon，`current` 的版本已更新

> **語法提醒**：`..` 雙點語法是比較 Branch 時的常見寫法，與 `git diff 1970s current` 等價。

### 練習 2：比較兩個 Branch（指定檔案）

```bash
git diff 1970s..current -- queen.txt
```

只顯示 `queen.txt` 的差異。這在大型專案中特別有用——當你只關心某個特定模組的變化時，不需要被其他不相干的變更淹沒。

### 練習 3：比較 HEAD 與前一個 Commit

```bash
git log --oneline
# 顯示 commit 歷史
git diff HEAD~1..HEAD
# 或
git diff HEAD HEAD~1
```

`-` 和 `+` 的意義，取決於你把哪個當作 A（舊）、哪個當作 B（新）。`HEAD~1` 是 HEAD 的父 Commit，也就是「上一個 Commit」。

### 練習 4：理解 Staged vs. Unstaged 的差異

這是最關鍵的實作環節。我們故意讓兩個檔案處於**不同的狀態**：

**情境設定**：
1. 修改 `queen.txt`：將 `Adam Lambert` 替換為 `Colt Steele`，`git add` 這個檔案（**已 Staged**）
2. 修改 `fleetwoodmac.txt`：將 `Stevie Nicks` 改為 `Stevie Chicks`，**不執行 `git add`**（**未 Staged**）

**實作指令與預期輸出**：

| 指令 | 顯示內容 | 原因 |
|------|----------|------|
| `git diff` | 只有 `fleetwoodmac.txt` 的變化（Stevie Nicks → Stevie Chicks）| 只顯示**未 Staged** 的變化 |
| `git diff --staged` | 只有 `queen.txt` 的變化（Adam Lambert → Colt Steele）| 只顯示**已 Staged** 的變化 |
| `git diff HEAD` | **兩個檔案**的變化都顯示 | 不論是否 Staged，顯示自上次 Commit 以來的所有變化 |

> **核心觀念**：這三個指令的差異，在於「比較的兩端」不同：`git diff` 比 Staging Area 與 Working Directory，`git diff --staged` 比 HEAD 與 Staging Area，`git diff HEAD` 比 HEAD 與 Working Directory。

### 常見錯誤與除錯

**錯誤：忘記順序導致誤解符號意義**
- `git diff 1970s..current`：File A = `1970s` 分支，`+` = 在 `current` 中新增的內容
- 如果你搞反了順序，會看到 `-` 變成新增、`+` 變成刪除，內容雖然正確但心理上會造成困惑

**錯誤：對未追蹤檔案執行 `git diff`**
- 未透過 `git add` 加入追蹤的檔案，`git diff` 不會顯示它們的變化
- 解決方式：用 `git diff HEAD` 即可看到新增檔案的內容（Git 會標記為從 `dev/null` 新增）

## 💡 重點摘要

- `git diff`：只顯示**未 Staged** 的 Working Directory 變化
- `git diff --staged`：只看**已 Staged**（準備 Commit）的變化
- `git diff HEAD`：不看 Staging 狀態，顯示**自上次 Commit 以來**的所有變化
- Branch 比較語法：`git diff branchA..branchB`，順序翻轉 `+`/`-` 的意義
- Commit 比較語法：`git diff HEAD~1..HEAD`，可替換為任意兩個 Commit Hash
- `--`（雙破折號）在 Git 指令中用來**分隔選項與檔案路徑**，是安全的寫法

## 關鍵字

git diff, git diff --staged, git diff HEAD, branch comparison, commit comparison, staging area, working directory, git switch, HEAD, HEAD~1, remote branch
