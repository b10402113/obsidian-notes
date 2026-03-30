# PR Conflict 處理——在命令列解決合併衝突

## 📝 課程概述

本單元處理一個在協作中無法避免的問題：**Pull Request 產生合併衝突（Conflict）**。當 PR 可以自動 merge 時，GitHub 會幫你處理；但當同一行程式碼被兩個人在不同 branch 上修改時，你就必須手動解決衝突。我們將學習如何透過命令列（而非 GitHub 的瀏覽器介面）完成這個流程。

## 核心觀念與實作解析

### 為什麼 PR 會有 Conflict？

PR 的 conflict 本質上跟 `git merge` 的 conflict 完全相同：

> 當 Git 嘗試將兩個 branch 合併時，如果同一行程式碼在兩個 branch 上有不同的修改，Git 無法自動判斷「誰的版本是正確的」，這時就會產生 conflict。

在 PR 的情境下：

```
main branch（由 Colt 直接編輯）：
  - 標題從 "Hello, world" 改為 "Hello there Everyone!!!"

new-heading branch（由 Stevie 的 feature）：
  - 標題改為 "Bock Bock"
```

GitHub 嘗試合併時，**同一行程式碼**同時被兩個 branch 修改了，PR 就顯示 "Can't automatically merge"。

### 解決 Conflict 的核心策略

**不要在 GitHub 瀏覽器上解決複雜的 conflict。** 雖然 GitHub 提供了瀏覽器端的 conflict 解決工具，但當 conflict 涉及多個檔案、多人複雜變更時，在命令列上解決會更安全且更有效率。

**標準流程（推薦）：**

```
1. Fetch PR 的 branch 到本地
2. 在那個 feature branch 上 merge main（不是反過來）
3. 解決衝突 → add → commit
4. Push feature branch（PR 自動更新）
5. 切換到 main，merge feature branch
6. Push main
```

### 實作步驟詳解

**步驟 1：Fetch 並切換到 PR 的 branch**

```bash
$ git fetch origin
# Git 回報：有新的 remote branch 'new-heading'

$ git switch new-heading
# Git 自動建立追蹤分支，並設定 upstream 到 origin/new-heading
```

**步驟 2：在 feature branch 上 merge main**

```bash
# 確保 feature branch 包含 main 的最新變更
$ git switch new-heading
$ git merge main
# ✗ 這裡會產生 conflict（因為 main 已經包含了 Colt 的修改）
```

**步驟 3：編輯衝突檔案，解決 conflict**

```html
<!-- 解決衝突：我們同時保留 Bootstrap 的 display-1 class 與 Stevie 的 Bock Bock -->
<h1 class="display-1">Bock Bock Everyone!!!!!</h1>
```

```bash
$ git add index.html
$ git commit -m "resolve heading conflict"
```

**步驟 4：Push feature branch，PR 自動更新**

```bash
$ git push origin new-heading
# GitHub 自動偵測：PR 的 feature branch 已經包含 main 的變更
# PR 頁面從 "Has conflicts" 變為 "Able to merge"
```

**步驟 5：在 main 上執行最終 merge**

```bash
$ git switch main
$ git pull origin main           # 先確保 local main 最新
$ git merge new-heading         # 這次 merge 沒有衝突
# --no-ff 參數可以強制產生 merge commit（GitHub 建議）
$ git push origin main
```

### `--no-ff` 參數的意義

```bash
$ git merge new-heading --no-ff
```

**`--no-ff`（no fast-forward）** 強迫 Git 即使可以 fast-forward，也要建立一個 merge commit。

> **為什麼要這樣做？** 因為在 PR 的流程中，我們希望歷史明確標記「這裡有一個 feature branch 被合併進來了」。如果用 fast-forward，歷史看起來就像一條直線，完全看不出分支的存在。

### GitHub 的建議流程（View Command Line Instructions）

當 PR 有 conflict 時，GitHub PR 頁面會提供具體的 CLI 指令：

```bash
git fetch origin
git checkout new-heading
git merge main
# 解決衝突
git push origin new-heading
```

這個流程跟我們上面描述的完全一致：**把 main merge 進 feature branch，解決衝突後 push，GitHub 會自動更新 PR。**

## 💡 重點摘要

- **PR Conflict = `git merge` Conflict**，兩者的根本原因完全相同，都是同一行程式碼被不同 branch 以不同方式修改。
- **在 feature branch 上 merge main**：這是解決 PR conflict 的標準做法，不是把 feature merge 回 main。
- **解決衝突後 push feature branch**：PR 會自動更新，不需要在 GitHub 介面上做任何操作。
- **`--no-ff`** 在 PR 流程中很有用——它強迫產生 merge commit，讓歷史明確記錄「這裡有一個 feature 被合併」。
- **不要在 GitHub 瀏覽器上解決複雜 conflict**：命令列解決衝突更安全，因為你有完整的編輯工具與本地的所有歷史。

## 關鍵字

Pull Request Conflict, Merge Conflict, git fetch, git merge main, Conflict Resolution, --no-ff, Feature Branch, Base Branch, Remote Tracking Branch, Auto-updating PR, GitHub Conflict Resolution, git checkout, git push origin
