# Merge Feature Branch 的三種策略

## 📝 課程概述

本單元探討當 feature 完成後，如何安全地將 feature branch merge 回 main branch。我們會比較三種不同的合併策略：自由合併（Free Merge）、討論後合併（Discussion then Merge）、以及 Pull Request（PR）。理解每種策略的適用情境，能幫助你根據團隊規模與專案性質選擇最合適的流程。

## 核心觀念與實作解析

### 為什麼「何時合併」是個值得討論的問題？

在 Feature Branch Workflow 中，**合併是團隊協作最後一步，也是最容易出錯的一步**。如果在沒有任何溝通與審查的情況下讓任何人隨時 merge，main branch 很快就會變得跟 Centralized Workflow 一樣混亂。

### 策略一：Free Merge（自由合併）

任何人在任何時候都可以直接 merge 自己的 feature branch 到 main。

```bash
$ git switch main
$ git pull origin main
$ git merge pricing-table
$ git push origin main
```

**適用情境**：極小型團隊（1-2 人）、內部 hobby 專案，或是大家都在同一個房間可以即時溝通時。

**問題**：沒有任何把關機制，任何人都可以把壞掉的程式碼 merge 進 main，破壞整個團隊的 stable branch。

### 策略二：Discussion then Merge（討論後合併）

Merge 前先透過 Slack、Email 或任何溝通工具，告知團隊你要 merge 了，確保沒有人反對。

**適用情境**：小型團隊（3-5 人），彼此熟悉但缺乏正式程式碼審查流程的專案。

**問題**：缺乏系統性的追蹤，討論內容無法留存歷史記錄；如果團隊成員時區不同或回覆不及時，流程容易卡住。

### 策略三：Pull Request（PR）

透過 GitHub 等平台的 PR 介面正式提出合併請求，經過審查、討論、反覆修改後，最後才 merge。

```bash
# 推送 feature branch
$ git push origin feature-branch

# 在 GitHub 介面上點擊 "Compare & pull request"
# 填寫 PR 標題與說明
# 等待審查者approve
# 合併進 main
```

**適用情境**：中大型團隊、有程式碼審查制度、或任何需要「多人同意才能變更 main」的專案。

### 合併時的實際操作順序

```bash
# 1. 切換到 main
$ git switch main

# 2. 確保 main 是最新狀態
$ git pull origin main

# 3. 合併 feature branch
$ git merge feature-name

# 4. 如果是 fast-forward（兩條分支沒有分歧），就直接前進
#    如果有分歧，會自動產生 merge commit

# 5. 推送 main
$ git push origin main

# 6. 團隊其他成員 pull 最新 main
# 其他成員在各自機器上：
$ git switch main
$ git pull origin main
```

### Merge Commit 與 Fast-Forward

合併時有兩種可能的結果：

| 合併類型 | 發生時機 | 歷史記錄 |
|---------|---------|---------|
| **Fast-forward** | Feature branch 沒有額外分歧，main 直接前進到 feature branch 的位置 | 乾淨的一直線 |
| **Merge commit** | Feature branch 和 main 都有新的 commit，產生了分歧 | 歷史中出現分支與合併節點 |

Fast-forward merge 是最理想的情況——歷史看起來像一條完美的直線。但如果你在 feature branch 上作業的期間團隊其他人也有新的 commit merge 到 main，就會形成「分歧」，這時 Git 會自動建立一個 merge commit 來記錄這次合併。

## 💡 重點摘要

- **Free Merge 只適合極小型團隊**，缺乏把關機制，main branch 的穩定性完全靠成員自律。
- **Discussion then Merge** 是介於自由與嚴謹之間的過渡方案，適合有溝通文化但缺乏正式流程的團隊。
- **Pull Request 是目前業界標準**——它把「誰能 merge」「誰應該 review」「討論內容是什麼」全部記錄下來，形成可追溯的歷史。
- **合併前務必 pull main**：確保你是在最新的基礎上合併，避免引入過時的變更。
- **合併後刪除 feature branch**：`git branch -d <branch-name>` 保持 local 乾淨；GitHub 上也可以刪除 remote branch，避免累積大量已合併的 stale branch。

## 關鍵字

Merge Strategy, Free Merge, Discussion then Merge, Pull Request, Fast-forward Merge, Merge Commit, Conflict, git switch main, git pull origin main, git merge, git branch -d, Feature Branch Lifecycle, Code Review
