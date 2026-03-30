# Comparing Changes Across Commits — Commit 與 Commit 之間的差異比較

## 📝 課程概述

本節介紹如何使用 `git diff` 比較任意兩個 Commit 之間的變化，不僅限於 `HEAD`。當你需要回顧某次重構究竟改了什麼，或是要分析一個長達數週的功能開發過程時，這個技巧會非常實用。語法與 Branch 比較非常相似，只是把分支名稱換成了 Commit Hash。

## 核心觀念與實作解析

### 基本語法

```bash
git diff <commit-hash-A>..<commit-hash-B>
```

和 Branch 比較一樣，**順序決定了 `+` 和 `-` 的含義**：

- `git diff abc123..def456`：abc123 是 File A（`master` 方向），def456 是 File B（`feature` 方向）
- `git diff def456..abc123`：**符號意義完全翻轉**

> **實用技巧**：如果覺得複製 Commit Hash 很麻煩，可以搭配 `git log --oneline` 先查詢歷史，再複製前幾個字元即可。

### 與 Branch Diff 的類比

Branch Diff（`git diff master..feature`）和 Commit Diff（`git diff abc123..def456`）在語法結構上完全一致，只是比較的對象從「分支」換成了「具體的 Commit 快照」。

兩者的核心邏輯相同：
- 都是計算**兩組檔案內容之間的差異集**
- 都服從**同一個 Diff 輸出格式**（`diff --git`、`@@`、`+`、`-`）
- **順序都會翻轉符號意義**

### 不連續 Commit 的差異

`git diff` 不要求你比較的兩個 Commit 是**連續的**。例如：

- Compare Commit A（第 3 個 Commit）與 Commit E（第 7 個 Commit）
- Git 會告訴你這段時間內**所有檔案的所有變化之和**
- 期間所有的中間 Commit（Commit D、F 等）都會被綜合成一組差異呈現

```bash
git log --oneline
# e4f8a1d add blue and purple
# b3c2d1e add orange
# a1b2c3d add red
# 9f8e7d6 initial commit

git diff a1b2c3d..e4f8a1d
# 顯示 "add red" 到 "add blue and purple" 這段時間內的所有變化
```

### 如何有效使用 Commit Diff

1. **功能回顧**：某個功能上線後，想知道開發期間總共改了什麼
2. **Bug 溯源**：某個壞掉的 Commit 是從哪個版本開始引入的（結合 `git bisect` 使用）
3. **Code Review**：在 Pull Request 中對比 Feature Branch 的起點與終點
4. **教學展示**：用真實的 Commit 歷史演示某個重構的完整過程

### 結合檔案指定

Commit Diff 同樣可以指定只看某個檔案：

```bash
git diff abc123..def456 -- colors.txt
```

### `--staged` 與 Commit Diff 的互補關係

| 指令 | 使用時機 |
|------|----------|
| `git diff` | 想看 Working Directory 裡**尚未 staging** 的變化 |
| `git diff --staged` | 想預覽**即將 commit** 的內容 |
| `git diff HEAD` | 想看 Working Directory 相對於**上次 commit** 的所有變化 |
| `git diff <hashA>..<hashB>` | 想回顧**任意兩個歷史時間點**之間的變化 |

### GUI 工具的幫助

面對複雜的 Commit Diff，終端機輸出可能仍然顯得密密麻麻。這時 GUI 工具（如 Git Kraken、VS Code 的 GitLens、或 GitHub Desktop）可以提供**視覺化的 Diff 檢視**，讓你一眼看出哪些檔案被修改、每個修改的內容是什麼。講師在影片最後演示了 Git Kraken 的 Diff 視圖，切換 Staged/Unstaged 只需點擊，Branch 間的 Diff 也是如此——這對大型專案的 Code Review 特別有幫助。

## 💡 重點摘要

- `git diff <hashA>..<hashB>` 可以比較**任意兩個 Commit** 之間的變化，不限於相鄰的 Commit
- **順序翻轉符號意義**：`A..B` 與 `B..A` 的 `+` 和 `-` 意義相反
- Commit Diff 與 Branch Diff 語法相同，只是把分支名稱換成了 Commit Hash
- 結合檔案參數 `-- <file>` 可以只看某個檔案在兩個 Commit 之間的差異
- GUI 工具（如 Git Kraken）在複雜 Diff 場景下能大幅提升可讀性

## 關鍵字

git diff, commit hash, git log, commit comparison, branch diff, staged changes, unstaged changes, git diff HEAD, Code Review, Git Kraken
