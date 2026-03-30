# Practice With Git Push — 推送練習與 GitHub 介面介紹

## 📝 課程概述

本節是 `git push` 的實作練習課，並帶你認識 GitHub 網站上的 Repo 介面。我們會在一個本地的小說 Repo 上做多個 Commit，然後將不同分支分别推送到 GitHub，觀察 GitHub 上 Commit 歷史、檔案內容與分支切換的呈現方式。

## 核心觀念與實作解析

### 完整工作流程：編輯 → Add → Commit → Push

```bash
# 1. 編輯本地檔案
rm -rf mood-board     # 刪除某個資料夾

# 2. 將變更加入 Staging Area
git add .

# 3. 建立 Commit
git commit -m "delete mood board folder"

# 4. 推送到 GitHub
git push origin master
```

這個流程是 Git + GitHub 協作的核心循環，每次完成一個有意義的修改後就重複這四個步驟。

### 為什麼 GitHub 不會自動同步？

GitHub 只是「代管」你的 Repo，它不會主動檢查你本地有沒有新的 Commit。你必須明確執行 `git push`，Git 才會將本地的新 Commit 上傳到 GitHub。

### 在 GitHub 上檢視 Repo

推送成功後，GitHub 頁面提供以下功能：

- **切換分支**：點擊分支下拉選單，切换到 `empty` 或 `master`
- **Commit 列表**：點擊「N commits」可以看到所有 Commit，按時間排序
- **檢視單個 Commit**：點擊某個 Commit，可以看到：
  - 該 Commit 修改了哪些檔案
  - 每個檔案的具體變更（使用 Diff 格式）
  - Commit Message
  - 該 Commit 的 Hash
- **檢視檔案**：點擊檔案名稱可以看到當前分支的最新內容

### 推送一個全新分支

如果本地建立了一個新分支（如 `empty`），而 GitHub 上還沒有這個分支：

```bash
# 在 empty 分支上
git push origin empty
```

GitHub 會自動建立 `empty` 分支。此後在 GitHub 頁面的分支下拉選單中，你就能看到並切换到這個分支了。

### 典型練習流程

講師設計的練習展示了完整的多分支推送流程：

1. 在 `master` 分支刪除 `mood-board` 資料夾 → Commit → Push
2. 切换到 `empty` 分支
3. 在 `empty` 分支新增 `summary.txt` 檔案 → Commit
4. 推送 `empty` 分支 → GitHub 上出現 `empty` 分支

完成後，GitHub 上會同時存在 `master` 和 `empty` 兩個分支，各自擁有不同的 Commit 歷史。

### 為什麼需要多個分支？

開源專案或團隊專案通常不會直接在 `master`/`main` 上開發，而是：
- 每個功能或修復建立一個新分支
- 在分支上完成開發、測試
- Merge 到 `master`/`main`
- 根據需要推送部分或全部分支到 GitHub

這個流程在後續章節（Feature Branch Workflow）中會詳細說明。

## 💡 重點摘要

- `git push` 不會自動同步——每次新 Commit 後都需要手動 Push
- 推送新分支時，GitHub 會自動建立該分支
- GitHub 介面提供完整的 Commit 歷史檢視、檔案檢視與分支切換功能
- 多分支場景下，每個分支獨立推送，不需要一次推送所有分支
- Commit Message 在 GitHub 上清晰可見，是團隊溝通的重要工具

## 關鍵字

git push, GitHub interface, commit history, branch switching, master, main, empty branch, git add, git commit, remote branch
