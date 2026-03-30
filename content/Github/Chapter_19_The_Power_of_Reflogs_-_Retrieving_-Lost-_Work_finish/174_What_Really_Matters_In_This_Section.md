# Git Reflog 搶救指南：什麼是 Reflog 及其核心價值

## 📝 課程概述

本單元是 Chapter 19 的開場，帶你認識 `git reflog` 這個「平時用不到，但需要時能救命」的命令。我們會從底層理解 Git 為什麼要記錄這些資料——它在何處儲存？記錄了什麼？當你搞砸了 `rebase`、`reset`，甚至 commit 消失時，Reflog 就是那把找回一切的鑰匙。

## 核心觀念與實作解析

### 什麼是 Reflog？

**Reflog 是 Reference Log 的縮寫**，翻成白話就是「Git 的參考移動日誌」。講師特別強調，雖然名字聽起來很無聊，但千萬別被誤導——這東西在關鍵時刻非常有用。

那麼什麼是「Reference」？在 Git 的世界裡，**Reference 就是「指向 commit 的指標」**，你可以把它想成箭頭。在視覺化圖中常見的：
- **Branch Reference**：每個分支名稱都是一個 reference，指向該分支的最新 commit
- **HEAD**：用來標記「你目前在哪個 commit 或哪個分支」的指標

> 當你移動這些指標時——切換分支、做新 commit、做 reset——Git 都會默默記錄下來。Reflog 就是這些「移動」的流水帳。

### Reflog 存在哪裡？

打開 `.git` 目錄，你會看到一個 `logs/` 資料夾。這裡面每一個檔案都對應一種 Reference：

```
.git/logs/
  ├── HEAD          # HEAD 的所有移動紀錄
  ├── refs/heads/   # 每個分支的 tip 移動紀錄（master、develop...）
  └── refs/remotes/# 遠端分支的推送紀錄
```

講師在影片中用 VS Code 直接打開了這個文字檔，裡面是一行一行的人類可讀文字，**並非 binary 格式**。每一筆紀錄包含：時間戳、舊的 commit、新 commit，以及操作描述。

### Reflog 記錄了哪些事件？

**幾乎所有會動到 HEAD 或分支 tip 的操作，都會被記錄**，包括：

- `git switch` / `git checkout`（切換分支）
- `git commit`（分支 tip 前進）
- `git reset`（移動 branch reference）
- `git rebase`（刪除並重建 commit）
- `git push`（遠端 branch 更新）

講師在終端機上演示：當執行 `git switch turtle` 時，`HEAD` 的 reflog 立刻新增一筆：
```
checkout: moving from master to turtle
```

進入 detached HEAD 模式後再做 `git checkout <commit-hash>`，reflog 又記錄：
```
checkout: moving from turtle to 295d...
```

回到 master 時，reflog 再次寫入：
```
checkout: moving from <detached-head-commit> to master
```

**這就是 Git 在幕後持續追蹤你的一舉一動**。而這些資料平常你我不會去動它，但當出事時，它就是唯一的線索。

### 為什麼 Reflog 如此重要？

當你執行 `git log` 時，你看到的是「目前存在於專案歷史中的 commit graph」。但這個 graph 是會被修改的：

- `git reset --hard <commit>` 會把分支指標往回移，中間的 commit 瞬間從 `git log` 消失
- `git rebase` 會重建 commit，舊的 commit hash 全變了
- `git commit --amend` 會修改最後一筆 commit

這些 commit **並沒有真正從磁碟消失**，只是 Git 覺得「反正也沒人指著你了」，所以暫時不理你。而 reflog 記錄了「你曾經在過哪裡」，所以我們可以靠著 reflog 裡的 commit hash 找回那些失蹤的 commit。

> **記住：Reflog 是你個人的、本地的、完整的 Git 操作日誌。** 它不是專案歷史的一部分，而是你個人在這台機器上的操作軌跡。

### 與 Git Log 的根本差異

| | `git log` | `git reflog` |
|---|---|---|
| 記錄內容 | commit graph（祖先關係） | 所有 HEAD / branch 指標的移動 |
| 包含 checkout 事件 | ❌ | ✅ |
| 包含 reset / rebase | ❌（那些 commit 可能已消失） | ✅ |
| 範圍 | 全域專案歷史 | 本地個人操作軌跡 |
| 預設壽命 | 永久 | 約 90 天後自動清除 |

## 💡 重點摘要

- **Reflog 是 Git 在本地持續維護的「指標移動日誌」，存在 `.git/logs/` 目錄下，人類可直接閱讀。**
- **每一筆 HEAD 移動（切分支、commit、reset、rebase）都會被記錄下來，即使那些 commit 已從 `git log` 消失。**
- **當你「搞丟 commit」時，Reflog 通常是最後的救生索——它保存了那些 commit 的 hash。**
- **平常用不到，但出事時（誤 rebase、誤 reset）是救命工具，這就是它存在的意義。**
- **千萬不要手動編輯 `.git/logs/` 下的任何檔案，Git 會維護它，你只需要在需要時查閱。**

## 關鍵字

`git reflog`、`Reference`、`Reflog show`、`HEAD`、`branch reference`、`git reset`、`git rebase`、`.git/logs/`、commit hash、detached HEAD、recovery、救命工具
