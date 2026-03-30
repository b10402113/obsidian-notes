# Undoing Changes Exercise — 實作練習與指令總整理

## 📝 課程概述

本節是 Chapter 10 的實作練習課。我們透過一個以 Beatles 名曲「Yesterday」歌詞為背景的 Repository，完整練習本節所學的所有指令：Detached HEAD 的進入與離開、建立 Branch 來保存歷史時間點、`git restore` 還原檔案，以及 `git reset` 在保留變更的前提下移動 Commit。這個練習的價值在於把 Restore/Reset/Revert 三個容易混淆的指令，放在一個具體的情境中讓你實際操作。

## 核心觀念與實作解析

### 練習環境說明

這個練習使用一個名為 `yesterday-exercise` 的 Repository，裡面只有一個 `lyrics.txt` 檔案。Repository 的 Commit 歷史模擬了 Paul McCartney 創作「Yesterday」的過程：

1. **初期版本**：以「Scrambled Eggs」為主題的歌詞（四個 Commit）
2. **過渡階段**：將「Scrambled Eggs」改為「Yesterday」的過程（四個 Commit）
3. 最後達到完整的「Yesterday」歌詞版本

> **注意**：這個練習同樣需要用到 `git clone`，詳情會在 Chapter 11 的課程中介紹。目前只需按照練習說明執行即可。

### 練習一：進入 Detached HEAD 觀看歷史

```bash
git checkout <first-commit-hash>
```

這裡的目標是回到**第一個 Commit**，看看當時的歌詞內容。進入 Detached HEAD 狀態後，你可以四處觀看，但千萬不要直接在上面做 Commit。

**離開 Detached HEAD**：
```bash
git switch master
# 或
git switch -
```

### 練習二：回到特定 Commit 並建立新 Branch

```bash
git checkout <commit-hash>   # 進入 Detached HEAD
git switch -c scrambled-eggs   # 在這個歷史時間點建立新分支
```

**情境**：你發現「Scrambled Eggs」版本的歌詞是某個粉絲群體的重要收藏，想要保存起來。你在「完成原始 Scrambled Eggs 版本」的 Commit 時刻建立了 `scrambled-eggs` 分支。

完成後，你可以自由切換分支而不會遺失任何內容：
```bash
git switch master    # 回到主線，看「Yesterday」
git switch scrambled-eggs  # 看「Scrambled Eggs」版本
```

### 練習三：使用 `git restore` 放棄 Working Directory 的修改

```bash
git restore lyrics.txt
```

**情境**：你在 `master` 分支上不小心刪除了 `lyrics.txt` 的全部內容。這時你不應該用 `Command+Z`（那是編輯器的 Undo），而是使用 Git 指令：

```bash
git restore lyrics.txt    # 現代寫法（推薦）
# 或
git checkout HEAD -- lyrics.txt   # 傳統寫法
```

執行後，`lyrics.txt` 恢復到 HEAD 的狀態，所有未 Commit 的修改消失。

### 練習四：使用 `git reset` 撤銷 Commit 但保留變更

```bash
git reset <commit-hash>
```

**情境**：你在 `master` 分支上連續 Commit 了兩個關於「404 頁面」搞笑歌詞的版本（「begin 404 lyrics」和「finish 404 lyrics」）。現在你後悔了，想把這兩個 Commit 從 `master` 上移除，但**不刪除這些內容**。

**步驟**：
1. `git log --oneline` 找到你想回到的那個 Commit（`yesterday` 完整版的最終 Commit）
2. `git reset <hash>` → 這兩個 404 Commit 被刪除，但檔案內容仍然在 Working Directory
3. `git switch -c 404` → 在新分支上，這些內容繼續存在
4. 在 `404` 分支上 `git add` + `git commit` → 內容移到新分支
5. `git switch master` → 主分支恢復乾淨的「Yesterday」歷史

> **為什麼用 `reset` 而非 `revert`？** 因為這兩個 404 Commit **還沒有 Push**，只有你自己知道，用 `reset` 完全可以，不會影響任何同事。

### 實作過程中的關鍵命令

```bash
git checkout <hash>         # 進入 Detached HEAD
git switch -                 # 回到上一個分支
git switch -c scrambled-eggs  # 建立並切換到新分支
git restore lyrics.txt        # 還原檔案（discard changes）
git reset <hash>             # 撤銷 Commit，保留變更在 Working Directory
git switch -c 404            # 建立新分支
git add lyrics.txt           # 將變更加入 Staging Area
git commit -m "write 404 parody lyrics"  # 提交
```

### `reset` 的 `--hard` 版本：這個練習為什麼不用

講師特別說明了這個練習為什麼**不用 `--hard`**：

- `git reset --hard` 不只刪除 Commit，**還會清除 Working Directory 的內容**
- 但這個練習的目標是「把錯誤的 Commit 移到新分支」，你需要保留那些內容
- 因此用**普通 `reset`**（不加參數）是最正確的選擇

## 💡 重點摘要

- Detached HEAD 可以用來「純觀看」歷史；`git switch -` 快速回到上一個分支
- `git switch -c <branch-name>` 可以在 Detached HEAD 狀態下建立並切換到新分支，保存歷史時間點
- `git restore <file>` 讓檔案回到 HEAD 狀態，是「discard changes」的推薦方式
- `git reset <hash>` 只刪除 Commit，**保留 Working Directory 的變更**；`--hard` 才會清除變更
- 將錯誤 Commit 移到新分支的標準流程：`reset`（保留變更）→ `switch -c <new-branch>` → `commit`
- 還沒 Push 的錯誤 Commit → `reset`（安全）；已經 Push 的 → `revert`（安全）

## 關鍵字

git checkout, detached HEAD, git switch, git switch -, git restore, git reset, git reset --hard, git commit, git log, scrambled-eggs, 404 lyrics, collaboration safety
