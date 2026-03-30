# Viewing Working Directory Changes — git diff 基本用法

## 📝 課程概述

本節介紹 `git diff` 最基礎的幾種用法：`git diff`（只看未 staging 的變化）、`git diff HEAD`（看所有自上次 Commit 以來的變化，含 staging 與否），以及 `git diff --staged`（只看已 staging 的變化）。掌握這三種變化，就足夠應付日常大多數 Diff 需求。

## 核心觀念與實作解析

### 先行準備：建立簡單的 Git 練習環境

講師建立了一個名為 `colors.txt` 的極簡 Repo，逐步加入 `red`、`orange`、`yellow`、`green`、`blue`、`purple` 等顏色，每個 Commit 只做一件小事。這樣的練習環境能幫助我們聚焦在 `git diff` 的邏輯上，而不被複雜的專案結構干擾。

### `git diff`（不帶任何參數）— 僅顯示未 staging 的變化

```bash
git diff
```

**用途**：比較「Staging Area」與「Working Directory」之間的差異。

也就是說，它告訴你：**「目前有哪些變化還沒被 `git add`？」**

實驗過程：
1. 開頭 `colors.txt` 還沒有任何變化，`git diff` 輸出空白
2. 刪除 `purple`，替換成 `indigo`，`git status` 顯示 `colors.txt` 已修改但未 staging
3. 此時 `git diff` 就會顯示這次的變化

**輸出解讀**：
- `--- a/colors.txt`：Staging Area 知道的版本（`purple` 還在）
- `+++ b/colors.txt`：Working Directory 中的版本（`indigo` 出現）
- `-purple`：這行來自 Staging Area 的版本，現在不存在了
- `+indigo`：這行來自 Working Directory 的版本，是新加的

> **關鍵理解**：`git diff` 不帶參數時，只顯示「尚未 `git add` 的變化」。一旦你 `git add` 了，該檔案的變化就從 `git diff` 的輸出中消失——因為現在 Staging Area 和 Working Directory 已經同步了。

### `git diff HEAD` — 顯示自上次 Commit 以來的所有變化

```bash
git diff HEAD
```

**用途**：比較「上次 Commit（HEAD）」與「目前 Working Directory」之間的全部差異，**無論是否已 staging**。

這裡的 `HEAD` 是一個指標，指向 repository 目前分支上的最新 Commit。

實驗過程：
1. 當 `colors.txt` 的變化**未 staging** 時：`git diff` 和 `git diff HEAD` 輸出相同
2. 執行 `git add colors.txt`（staging 變化）後：
   - `git diff`：**空白**（Staging Area 和 Working Directory 已同步）
   - `git diff HEAD`：**仍顯示變化**（因為 HEAD 仍指向舊 Commit）

這就是兩者最核心的區別：
- `git diff` → Staging Area vs. Working Directory
- `git diff HEAD` → HEAD（上次 Commit）vs. Working Directory（含 Staging Area）

### `git diff --staged`（或 `--cached`）— 只看已 staging 的變化

```bash
git diff --staged
# 或
git diff --cached
```

**用途**：比較「上次 Commit（HEAD）」與「Staging Area」之間的差異。

也就是說，它回答的是：「如果現在執行 `git commit`，實際上會提交什麼內容？」

實驗過程：
1. `numbers.txt` 已 staging（`git add numbers.txt`），`colors.txt` 未 staging
2. `git diff --staged` → 只顯示 `numbers.txt` 的變化
3. `git diff` → 只顯示 `colors.txt` 的變化
4. `git diff HEAD` → 兩者的變化都顯示

> **注意**：`--staged` 和 `--cached` 在功能上完全相同，只是名稱不同。講師個人偏好 `--staged`，因為更能反映「Staging Area」的概念。`cached` 這個名稱來源於 Git 內部文檔，比較不容易聯想。

### 實用情境對照表

| 指令 | 比較的兩端 | 回答的問題 |
|------|-----------|------------|
| `git diff` | Staging Area vs. Working Directory | 有什麼變化還沒 staging？ |
| `git diff HEAD` | HEAD vs. Working Directory（含 Staging）| 自上次 Commit 後，所有變化是什麼？ |
| `git diff --staged` | HEAD vs. Staging Area | 準備要 commit 的內容是什麼？ |

### 新增未追蹤檔案時的 `git diff HEAD`

如果用 `touch numbers.txt` 建立一個全新的未追蹤檔案：
- `git diff`：**不會顯示**（未追蹤檔案不在比較範圍內）
- `git diff HEAD`：**會顯示** `numbers.txt` 是新增的（因為它在 HEAD 中不存在，現在存在了）

Git 會用 `dev/null` 表示「這個檔案在舊版本中根本不存在」，所以你只會看到「新增」的內容，不會有「刪除」的部分。

## 💡 重點摘要

- `git diff` 只顯示**未 staging** 的變化；一旦 `git add` 後就消失
- `git diff HEAD` 是最強大的形式，**所有變化無所遁形**，無論是否已 staging
- `git diff --staged`（或 `--cached`）讓你預覽「即將被 commit 的內容」
- 新建的未追蹤檔案，`git diff` 不會顯示，但 `git diff HEAD` 會標記為新增
- 這三個指令的差別在於「比較的兩端」不同，理解「比較的範圍」是關鍵

## 關鍵字

git diff, git diff HEAD, git diff --staged, git diff --cached, staging area, working directory, HEAD, tracked files, untracked files, git add
