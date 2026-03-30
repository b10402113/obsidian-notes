# Git Pull & Merge Conflicts — 取回並合併的完整流程

## 📝 課程概述

本節介紹 `git pull`——從遠端取回 Commit 並**直接合併到本地分支**的指令。`git pull` 等於 `git fetch` 加上 `git merge`，會將遠端的變更自動合併到你的 Working Directory。由於合併可能產生衝突，本節也詳細說明了如何處理 Pull 時的 Merge Conflict。

## 核心觀念與實作解析

### `git pull` 的運作方式

```
git pull = git fetch + git merge
```

Git 執行 `git pull` 時，會：
1. **Fetch**：與 Remote 通信，下載新 Commit，更新 Remote Tracking Branch
2. **Merge**：自動將 Remote Tracking Branch 的新 Commit 合併到**你目前所在的本地分支**

> **Pull 的語法**：`git pull <remote> <branch>`——你在哪個分支，就 Pull 到哪個分支。

### Pull 的語法

```bash
# 基本語法：從指定 Remote 的指定分支 Pull
git pull origin main

# 省略語法（常用）：自動使用當前分支的 Upstream
git pull
```

### 為什麼可以省略參數？

當本地分支設定了 Upstream（追蹤關係）時：
- `git pull` 自動使用 `origin/main`（如果你在 `main` 分支）
- `git pull` 自動使用 `origin/food`（如果你在 `food` 分支）

這就是為什麼首次推送新分支時用 `git push -u origin branch-name` 那麼重要——它設定了追蹤關係，讓未來所有操作都更簡潔。

### Fetch + Merge vs. 直接 Pull

| 操作 | Remote Tracking Branch 更新 | Working Directory 更新 | 風險 |
|------|------------------------|---------------------|------|
| `git fetch` | ✅ | ❌ | 安全 |
| `git pull` | ✅ | ✅（自動 Merge）| 可能有衝突 |

### Merge Conflict 處理流程

**情境**：你在本地修改了 `coffee.txt`，同時隊友也在 GitHub 上修改了 `coffee.txt`，你執行 Pull：

```bash
git pull origin food
# 輸出：Auto-merging coffee.txt
# 輸出：CONFLICT (content): Merge conflict in coffee.txt
```

Git 無法自動合併，會報告衝突。處理步驟：

1. **開啟衝突檔案**，手動解決衝突（決定保留哪個版本，或合併兩者）
2. **標記為已解決**：`git add coffee.txt`
3. **完成 Merge Commit**：`git commit`（Git 已預填 Merge Commit 訊息）

### 實際操作：解決衝突

Git 在衝突檔案中標記衝突範圍：

```html
<<<<<<< HEAD
<!-- 這是你本地的版本 -->
<div>My Coffee</div>
=======
<!-- 這是遠端的版本 -->
<div>Colleague's Coffee</div>
>>>>>>> abc1234 (remote version)
```

解決方式：
- 保留本地：`刪除 <<< === >>>`，只留本地方
- 保留遠端：只留遠端版本
- **合併兩者**：同時保留兩個版本的內容

### 為什麼要先 Pull 再 Push？

講師強調的最佳實踐：**Push 之前先 Pull**。

如果你直接 Push，但 GitHub 上已有新 Commit：
- Git 會**拒絕 Push**（非 Fast-forward）
- 你需要先 Pull、解決衝突、再 Push

```bash
git pull origin main   # 先 Pull
git push origin main  # 再 Push
```

### Fetch + Merge 分步進行的好處

如果你不想直接 Pull（因為會自動合併），可以分步進行：

```bash
git fetch origin main          # 只下載，不合併
git log HEAD..origin/main      # 預覽有哪些新 Commit
git merge origin/main          # 手動決定合併
```

這樣的好處是：在 Merge 之前，你可以先清楚看到即將合併的內容是什麼。

## 💡 重點摘要

- `git pull` = `fetch` + `merge`，直接將遠端變更合併到本地分支
- `git pull` 可以省略 Remote 和 Branch 參數（當有設定 Upstream 時）
- Pull 時如果同一檔案同時被本地和遠端修改，會發生 Merge Conflict
- Merge Conflict 的處理流程：手動解決 → `git add` → `git commit`
- **最佳實踐**：Push 之前先 Pull，避免被 Git 拒絕
- 如果不想直接合併，先 `git fetch`，確認內容後再 `git merge origin/main`

## 關鍵字

git pull, git fetch, git merge, merge conflict, git pull origin main, git pull (shorthand), upstream branch, tracking, fast-forward, resolve conflict, git add, git commit
