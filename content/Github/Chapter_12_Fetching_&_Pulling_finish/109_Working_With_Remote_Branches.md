# Working With Remote Branches — 如何操作遠端分支

## 📝 課程概述

本節說明當 GitHub 上有多個分支，而你的本地只有一個時，該如何**把遠端分支取下來當作本地分支工作**。透過 `git switch` 和 `git checkout`，你可以輕鬆地將任何 Remote Tracking Branch 轉換為一個可自由開發的本地分支，並自動建立追蹤關係。

## 核心觀念與實作解析

### 為什麼 Clone 後本地只有一個分支？

Clone 時，Git 只會把**預設分支**（`main` 或 `master`）Checkout 出來，成為你的本地分支。**其他分支只存在 Remote Tracking Branch**（如 `origin/food`、`origin/movies`），GitHub 上有，但你的本地沒有對應的本地分支。

### 如何取得並工作在遠端分支上？

#### 方式一：使用 `git switch <branch-name>`

如果 GitHub 上有一個 `food` 分支，而你的本地只有 `main`：

```bash
git switch food
# 輸出：Branch 'food' set up to track remote branch 'food' from origin.
# Switched to a new branch 'food'
```

**Git 的自動偵測邏輯**：
- 你說要切換到 `food` 分支
- Git 檢查發現本地沒有 `food` 分支
- Git 同時檢查是否存在 `origin/food` Remote Tracking Branch
- 如果存在，Git 自動建立一個本地 `food` 分支，並設定它**追蹤** `origin/food`

這就是 `git switch` 比舊式的 `git checkout -b` 更方便的地方——**你不需要手動建立分支並指定追蹤關係**，Git 會自動幫你處理。

#### 方式二：使用 `git checkout -b`（舊式做法）

```bash
git checkout -b food origin/food
```

`-b` 參數建立新分支，`origin/food` 指定以哪個 Remote Tracking Branch 為基礎。

### 追蹤關係（Tracking）的意義

當一個本地分支設定了追蹤關係（Tracking），意味著：

- **Push**：可以省略 Remote 和 Branch 名稱，直接 `git push`
- **Pull**：可以省略 Remote 和 Branch 名稱，直接 `git pull`
- Git 會記住「這個本地分支 <-> 那個遠端分支」的對應關係

> **預設分支的追蹤關係是自動設定的**。Clone 時，`main` 會自動追蹤 `origin/main`，你不需要做任何事情。

### 實作：將遠端新分支取下來

情境：隊友在 GitHub 上建立了一個新分支 `bugfix`，並告訴你「快去看看！」

```bash
# 1. 先 Fetch，確保 Remote Tracking Branch 更新
git fetch

# 2. 切換到該分支（Git 會自動建立並設定追蹤）
git switch bugfix

# 3. 確認追蹤關係
git branch -vv
# 輸出：bugfix abcd123 [origin/bugfix] Add bug fix description
```

### 如何查看遠端分支的存在？

Clone 之後，如果有人在 GitHub 上建立了新分支：

```bash
git branch -r    # 不會馬上看到，因為還沒 Fetch
git fetch        # 與 GitHub 同步，更新 Remote Tracking Branch
git branch -r    # 現在可以看到新分支了
```

> **重要**：Clone 後，你的本地 Repo **不會自動知道** GitHub 上新增的分支。必須執行 `git fetch`，Git 才會去問 GitHub「有新分支嗎？」，並更新 Remote Tracking Branch。

## 💡 重點摘要

- `git switch <branch-name>` 會自動偵測 `origin/<branch-name>` 是否存在，並自動建立追蹤關係
- `git fetch` 後，所有 GitHub 上新建立的分支才會出現在 Remote Tracking Branch 清單中
- **追蹤關係**讓未來的 Push 和 Pull 可以省略 Remote 和 Branch 參數
- 如果本地沒有某個分支，只需 `git switch <name>`，Git 就會自動幫你建立並設定追蹤
- `git branch -vv` 可以查看本地分支與遠端分支的追蹤關係

## 關鍵字

git switch, git checkout -b, tracking, git branch -vv, git fetch, origin/food, origin/main, remote branch, local branch, upstream branch
