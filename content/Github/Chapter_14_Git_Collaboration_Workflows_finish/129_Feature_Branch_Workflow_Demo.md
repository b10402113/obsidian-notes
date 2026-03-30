# Feature Branch Workflow 實作演示

## 📝 課程概述

本單元透過 Colt 和 Stevie 的實際操作，完整演示 Feature Branch Workflow 的日常流程：從建立分支、在分支上開發、push 到 GitHub、fetch 團隊成員的分支、在同一個分支上協作，最後 merge 回 main。我們將體會這種工作流程如何讓協作變得安全且有條理。

## 核心觀念與實作解析

### 從 main 分支出來之前：先確認是最新狀態

在建立新的 feature branch 之前，一個好的習慣是先確認本地的 main 是最新的：

```bash
$ git switch main
$ git pull origin main      # 確保 main 包含團隊所有人的最新 commit
$ git switch -c navbar     # 再從最新的 main 建立新分支
```

> **為什麼這麼做？** 如果你在落後的 main 上建立分支，你的 feature branch 從一開始就缺少了團隊其他成員的 commit，merge 回 main 時衝突會特別多。

### Stevie 的情境：建立 navbar branch

```bash
$ git switch -c navbar      # 建立並切換到 navbar branch
# 在 navbar branch 上作業：複製 Bootstrap Navbar 的程式碼
$ git add index.html
$ git commit -m "add broken navbar"
$ git push origin navbar    # 把分支 push 到 GitHub，main 仍然乾淨
```

**這裡的關鍵認知**：即使 Navbar 程式碼有 Bug、無法正常運作，直接 push 到 GitHub 也完全沒問題——因為那是 `navbar` branch，不是 `main`。

### Colt 的情境：建立 pricing-table branch

```bash
$ git switch main
$ git pull origin main       # 確認最新
$ git switch -c pricing-table
# 開發 pricing table 功能
$ git add index.html
$ git commit -m "add pricing table"
$ git push origin pricing-table
```

**Feature Branch 隨時可以 push**，不需要像 Centralized Workflow 一樣每次都先 pull 再 merge 再 push。

### Fetch 並 checkout 團隊成員的分支

Stevie 的 Navbar 有 Bug，Colt 想要幫忙修復。Colt 這樣做：

```bash
$ git fetch origin           # 查看 remote 有哪些新分支
# Git 回報：origin/navbar 是新的 remote tracking branch

$ git switch navbar         # Git 自動建立本地 navbar branch 並設定追蹤
# Branch 'navbar' set up to track remote branch 'navbar' from origin.
```

> **重要語法**：`git switch <branch-name>` 會自動在本地建立一個與 `origin/<branch-name>` 同名的 branch 並設定追蹤關係——不需要 `git checkout -b`，超級方便。

### 在同一個 feature branch 上協作

Colt 在 `navbar` branch 上找到了 Bug：**缺少 Bootstrap 的 JavaScript 檔案**。

```html
<!-- 在 index.html 底部加入 Bootstrap JS -->
<script src="bootstrap.bundle.min.js"></script>
```

```bash
$ git add index.html
$ git commit -m "fix navbar bug"   # 這是在 navbar branch 上的 commit
$ git push origin navbar           # 推送到 GitHub，Stevie 的 navbar branch 就更新了
```

Stevie 再把自己的分支 pull 下來：

```bash
$ git pull origin navbar
# Colts 的修復 commit 已經在 navbar branch 上了
```

**同一個 feature branch 可以同時容納多位開發者的 commit**，而且 main branch 完全不受影響。

### Merge 回 main

當 feature 完成後，merge 回 main 的流程：

```bash
$ git switch main
$ git pull origin main          # 確認 main 是最新狀態
$ git merge pricing-table       # 合併 pricing-table
# 順利的話：Fast-forward merge，無需 merge commit
$ git push origin main
```

完成後可以安全刪除已合併的 branch：

```bash
$ git branch -d pricing-table   # 刪除本地 branch
# 在 GitHub 介面上也可以刪除 remote branch
```

### 沒有 commit 數量限制，沒有 merge 焦慮

跟 Centralized Workflow 相比，Feature Branch Workflow 最大的解放是：**你想 commit 多少次都可以**，不需要等到「功能完整了」才能 push。commit 是你的私人進度保存點，只要 branch 還沒 merge 回 main，團隊其他人都看不到，也不會受到影響。

## 💡 重點摘要

- **建立 feature branch 前先 pull main**：確保新分支是基於最新狀態，減少未來 merge 的衝突。
- **`git switch <remote-branch>` 自動建立追蹤分支**：這是 fetch remote branch 最簡潔的方式。
- **Feature branch 是協作的場所**：多位開發者可以在同一個 feature branch 上 commit、push、pull，main 永遠乾淨。
- **找到 Bug？直接在同 branch 修復**：不需要離開 branch、不需要創造新的 branch，修復 commit 直接追加在 feature branch 上。
- **Feature 完成後的流程**：switch main → pull → merge → push → 刪除用完的 branch。保持 branch list 整潔。

## 關鍵字

git switch -c, git fetch origin, git switch <branch>, Remote Tracking Branch, Fast-forward Merge, git branch -d, Feature Complete, Main Pull, Tracking Relationship, Collaboration on Branch, git push origin, Stale Branch
