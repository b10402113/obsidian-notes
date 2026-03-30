# Introducing Git Push — 將本地倉庫推送到 GitHub

## 📝 課程概述

本節介紹 `git push`——將本地倉庫的分支與 Commit 歷史上傳到 GitHub 的指令。在設定好 Remote（`origin`）之後，`git push` 就是讓你的程式碼「被世界看到」的最終一步。我們會詳細說明 `git push` 的語法，並展示如何將多個分支分別推送到 GitHub。

## 核心觀念與實作解析

### `git push` 的基本語法

```bash
git push <remote> <branch>
# 例如：
git push origin master
git push origin main
git push origin feature-branch
```

- **`<remote>`**：你的遠端名稱（通常是 `origin`）
- **`<branch>`**：你要推送的本地分支名稱

### 推送後 GitHub 會發生什麼？

1. Git 將本地 `master`（或 `main`）分支的所有 Commit 上傳到 GitHub
2. GitHub 自動建立（或更新）對應的分支
3. 你可以在 GitHub 網站上看到所有 Commit 歷史、檔案內容與差異

> **重要**：GitHub 不會自動與你的本地 Repo 保持同步。每次你在本地做新的 Commit，都需要手動 `git push` 才能讓 GitHub 看到這些變更。

### 第一次推送會發生什麼？

當你第一次推送 `master` 分支時：
- GitHub 上**還沒有** `master` 分支
- `git push origin master` 會在 GitHub 上**建立** `master` 分支
- 這是因為 GitHub 從來沒有收到過這個分支的任何 Commit

### 推送後續 Commit

之後每次新增 Commit，只需要：
```bash
git push origin master   # 將新的 Commit 上傳到 GitHub
```

Git 會聰明地只上傳「本地有、GitHub 上沒有」的新 Commit，而不是整個 Repo 重新上傳。

### 推送多個分支

如果你有多個本地分支，可以分别推送：

```bash
git push origin master        # 推送 master
git push origin empty         # 推送 empty 分支
git push origin feature-login # 推送 feature-login 分支
```

每個分支是獨立推送的。GitHub 上可以看到所有推送過的分支。

### 推送任意分支到任意遠端分支

`git push` 的完整語法支援「本地分支 → 遠端分支」的對應：

```bash
git push origin local-branch:remote-branch
# 例如：
git push origin cats:master   # 將本地的 cats 分支推送到 GitHub 的 master 分支
```

這個語法很少使用（因為通常本地分支名稱與遠端相同），但理解它有助於深入了解 Remote 的運作原理。

### 在 GitHub 上查看推送結果

推送成功後，GitHub 頁面會顯示：
- **Commit 歷史**：所有 Commit，按時間排序
- **分支列表**：所有已推送的分支（點擊分支下拉選單切換）
- **檔案內容**：最新 Commit 的所有檔案
- **每個檔案的差異**：點擊某個 Commit 可看到該 Commit 修改了哪些檔案

> **常見問題**：如果你在 GitHub 上看不到最新的 Commit，**先確認你正在觀看正確的分支**。GitHub 預設顯示預設分支（通常是 `master` 或 `main`）。

## 💡 重點摘要

- `git push <remote> <branch>`：將指定的本地分支推送到遠端倉庫
- 第一次推送時，GitHub 會自動建立對應的分支
- 後續推送只上傳新 Commit，GitHub 不會自動同步
- `git push` 是 Git 的原生命令，適用於任何 Remote，不僅限於 GitHub
- 每次在本地做新的 Commit，都需要執行 `git push` 才能讓 GitHub 看到更新

## 關鍵字

git push, origin, master, main, remote branch, git push origin master, first push, GitHub repository, push multiple branches
