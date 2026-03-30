# Git Push -u (Upstream) — 設定上游分支的意義

## 📝 課程概述

本節介紹 `git push` 的 `-u`（或 `--set-upstream`）選項，以及背後的「上游分支（Upstream Branch）」概念。設定 Upstream 後，你就可以在未來直接執行 `git push`（不帶任何參數），而不需要每次都輸入完整的 `git push origin branch-name`。這個小小的便利在長期專案開發中會節省大量時間。

## 核心觀念與實作解析

### 什麼是 Upstream（上游分支）？

Upstream Branch 是本地分支與遠端分支之間的**連接關係**。把它想成是一條「預設路線」：

```
本地分支 master ←Upstream 連接→ 遠端分支 origin/master
```

設定了 Upstream 後，Git 會記住「這個本地分支應該對應到哪個遠端分支」，從此你就可以使用簡化的指令。

### `git push -u origin <branch>` 的意義

```bash
git push -u origin dogs
# 等價於
git push --set-upstream origin dogs
```

`-u` 的效果分為兩步：

1. **立即動作**：將 `dogs` 本地分支推送到 GitHub，建立 `dogs` 遠端分支
2. **永久設定**：Git 記住「日後在 `dogs` 分支執行 `git push` 時，自動推送到 `origin/dogs`」

### 設定 Upstream 後的差異

**沒有設定 Upstream**：
```bash
# 每次 Push 都必須指定 remote 和 branch
git push origin dogs
```

**已設定 Upstream**：
```bash
# 只需要執行 git push，Git 會自動使用已記住的 upstream
git push
```

### `git push` 失敗時的提示

如果在一個沒有設定 Upstream 的分支上執行 `git push`（不帶參數），Git 會失敗並給出提示：

```
fatal: The current branch dogs has no upstream branch.
To set the tracking information for this branch, run:

  git push --set-upstream origin dogs
```

這就是 Git 在告訴你：「你需要先設定 Upstream，才能使用簡化的 `git push`。」

### 為什麼不預設設定 Upstream？

預設情況下，`git push` 不知道你希望哪個本地分支對應到哪個遠端分支。Git 預設的行為是：
- 如果本地分支名稱在遠端已存在同名分支 → 自動推送到那個分支
- 如果本地分支名稱在遠端不存在 → **失敗**，需要明確指定或使用 `-u`

### Upstream 設定與 Pull 的關係

Upstream 不只影響 Push，也影響 Pull。設定了 Upstream 後：
- `git push`：自動推送到指定的遠端分支
- `git pull`：自動從指定的遠端分支拉取並 Merge

兩者共同組成了一個本地分支與遠端分支的「雙向通道」。

### 進階語法：指定不同的遠端分支名稱

一般在首次推送時：
```bash
git push -u origin dogs   # 本地 dogs → 遠端 origin/dogs
```

但也可以建立非對稱的對應關係（不常見）：
```bash
git push -u origin dogs:cats   # 本地 dogs → 遠端 origin/cats
```

> **警告**：非對稱的 Upstream 設定容易造成混淆，不建議在正規開發流程中使用。

## 💡 重點摘要

- `git push -u origin <branch>` 的 `-u`（`--set-upstream`）會設定該分支的 Upstream
- 設定 Upstream 後，未來在該分支上執行 `git push` 不需要任何參數
- Upstream 是本地分支與遠端分支之間的「預設連接」，同時影響 Push 和 Pull
- `git push` 失敗時，Git 會提示你需要設定 Upstream，並給出具體指令
- 建議在每次首次推送新分支時，都使用 `-u` 參數，省去未來每次重複輸入的麻煩

## 關鍵字

git push -u, --set-upstream, upstream branch, tracking, git push, git pull, origin, remote branch, git branch --set-upstream-to
