# Diffing Specific Files — 精確指定檔案與分支的 Diff 比較

## 📝 課程概述

本節介紹如何讓 `git diff` 的比較範圍更精確——不只看「所有檔案的變化」，而是鎖定特定檔案（`git diff -- <file>`）、同時指定分支比較（`git diff branchA..branchB`），以及如何結合這些技巧做到細粒度的變化追蹤。

## 核心觀念與實作解析

### 為什麼要指定檔案？

在大型專案中，一次 `git diff HEAD` 可能會顯示數十個檔案的變化。很難集中注意力在你想關注的那個修改上。這時，加上檔案路徑參數就能精確鎖定目標：

```bash
git diff HEAD -- style/main.css
# 或更簡潔
git diff HEAD style/main.css
```

這告訴 Git：「我只關心 `style/main.css` 這個檔案，請只顯示它的差異。」

> **實用場景**：假設你修改了 `index.html`、`main.css` 和 `grid.js` 三個檔案，但現在只想回顧 CSS 的變化——`git diff HEAD style/main.css` 就是最乾淨的方式。

### 一次指定多個檔案

用空格分隔多個檔案即可：

```bash
git diff HEAD style/main.css index.html
```

這對 `git diff --staged` 同樣有效：
```bash
git diff --staged -- numbers.txt
```

### `git diff` 比較兩個 Branch

Branches 是 Git 中用來隔離不同工作線的機制。當你需要知道「這個 Feature Branch 相比 `master`（或 `main`）多了什麼或少了什麼」時：

```bash
git diff master..feature-branch
# 或（兩者等價）
git diff master feature-branch
```

**順序很重要**：
- `master..feature-branch`：File A = `master`，File B = `feature-branch`
  - `+` = 在 `feature-branch` 中**新增**的內容
  - `-` = 在 `master` 中存在、但**不在** `feature-branch` 中的內容
- `feature-branch..master`：符號含義**完全翻轉**

實驗過程：
1. 建立 `odd-numbers` 分支，在上面刪除數字 `2` 和 `4`，加入數字 `5`
2. 執行 `git diff master..odd-numbers`：
   - `+` 顯示：新增的奇數內容（`1`、`3`、`5`）
   - `-` 顯示：被刪除的偶數內容（`2`、`4`）
3. 執行 `git diff odd-numbers..master`：符號翻轉，變成顯示偶數是「新增」的

> **為什麼順序重要？** 因為 Diff 是**有方向性**的。比較「過去 → 現在」和「現在 → 過去」，`+` 和 `-` 的意義剛好相反。在撰寫自動化腳本或撰寫 Commit Message 時，請特別留意這個順序。

### Branch Diff 的常見應用

- **Code Review**：在 Merge 之前，確認 Feature Branch 相比主線做了哪些改動
- **衝突評估**：Merge 前預估可能發生衝突的範圍
- **歷史對比**：比較不同時期的分支狀態

### 與 Staged/HEAD 選項的組合

`git diff` 的各個參數可以疊加使用。例如：

```bash
# 只看某分支上某檔案的 staging 變化
git diff --staged feature-branch -- main.css

# 只看某分支上某檔案相對於 HEAD 的所有變化
git diff feature-branch -- index.html
```

這個靈活性讓 `git diff` 能應對幾乎所有你能想到的比較需求。

## 💡 重點摘要

- 在 `git diff` 後面加上檔案路徑，可以將比較範圍精確鎖定到單一檔案或多個檔案
- 比較分支時，`branchA..branchB` 的**順序決定了 `+` 和 `-` 的含義**
- `git diff master feature` 與 `git diff master..feature` 在功能上等價
- 結合 `--staged` 或 `HEAD` 與檔案參數，可以做到極細粒度的變化追蹤
- Branch Diff 是 Merge 前做 Code Review 的標準做法

## 關鍵字

git diff, branch comparison, git diff branch, double dot syntax, HEAD, staging area, file path, feature branch, master, main
