# Pull Request 概念與實作

## 📝 課程概述

本單元深入介紹 Pull Request（PR）——這是一個**建立在 GitHub 等平台上，而非 Git 原生的協作機制**。PR 讓開發者可以正式請求將自己的 feature branch 合併進 main branch，並在此過程中接受團隊的程式碼審查、討論、修改，最後由擁有權限的人批准或拒絕合併。

## 核心觀念與實作解析

### PR 不是 Git 原生功能

**Pull Request 是 GitHub、GitLab、Bitbucket 等平台提供的功能，不是 Git 本身的命令。** 這意味著：

- 你無法在純 CLI 的 Git 環境中創建 PR
- PR 的行為（審查流程、權限設定、merge 按鈕）取決於你使用的平台
- 但 PR 的底層仍然是 `git merge`

### PR 的核心價值：為 merge 引入「控制閥」

當團隊有一定規模時，**「誰來批准合併」是個必須回答的問題**。PR 就是這個問題的標準答案：

> 「我想把這個 feature 合併進 main。我的程式碼有人檢查過嗎？有人同意嗎？測試通過了嗎？只有在這些條件都滿足的情況下，才允許合併。」

### PR 的典型生命週期

```
建立 PR → 團隊審查 → 討論與修改 → 批准 → 合併進 main
                  ↘ 或：被關閉（拒絕）
```

### 發起 PR 的實作步驟

**步驟 1：推送 feature branch 到 GitHub**

```bash
$ git push origin navbar
```

**步驟 2：在 GitHub 介面上建立 PR**

在 GitHub repository 頁面，點擊 **Compare & pull request** 按鈕，GitHub 會聰明地幫你：
- 自動選擇你的 branch 為 "compare"
- 自動選擇 main 為 "base"
- 顯示這個 branch 與 main 的 diff

**步驟 3：填寫 PR 資訊**

- **標題（Title）**：簡潔描述這個 PR 的目的
- **內容（Body）**：這個 PR 包含什麼變更？解決了什麼問題？為什麼需要這個 PR？

在大公司或開源專案中，PR 的內容格式通常有嚴格要求（例如必須包含：Summary、Test Plan、Related Issues）。

**步驟 4：@mention 團隊成員**

在 PR 內容中用 `@username` 可以通知特定成員前來審查。

### PR 上的協作互動

一旦 PR 開出來，審查者可以：

- **留下 Comment**：對某行程式碼或整體內容發表意見
- **要求修改（Request Changes）**：阻止 merge 直到問題被解決
- **批准（Approve）**：明確表示同意這個 PR 可以 merge
- **直接 merge**：如果權限允許，點擊 Merge 按鈕完成合併

開發者收到反饋後，可以在同一個 PR 上追加 commit（因為 PR 追蹤的是 branch，而不是 snapshot），不需要關閉舊 PR 開新 PR。

### PR Merge 完成後

```bash
# 合併完成後，本地 main 已落後於 GitHub，需要 pull
$ git switch main
$ git pull origin main

# 刪除已合併的 feature branch（可選，但建議保持乾淨）
$ git branch -d navbar
```

### 誰可以 merge？

PR 的核心價值之一是「**控制誰有權限合併**」。如果沒有設定限制，任何能 push 到 repository 的人都可以繞過審查直接 merge。

這就是下一個單元 **Branch Protection Rules** 要解決的問題。

## 💡 重點摘要

- **PR 是平台功能而非 Git 原生命令**——GitHub、GitLab、Bitbucket 都有各自的 PR 實作，但概念相同。
- **PR 為 merge 提供了透明、可追溯的審查機制**：所有討論、審查意見、修改記錄都保存在 PR 中，成為專案歷史的一部分。
- **PR 追蹤的是 branch**：在 PR 合併之前，追加的 commit 會自動反映在同一個 PR 中，不需要關閉重開。
- **@mention** 讓你可以精確通知特定團隊成員前來審查，提高協作效率。
- **Merge PR 不等於結束**：合併後團隊成員仍需 `git pull` 來更新本地的 main branch，並清理用完的 feature branch。

## 關鍵字

Pull Request, GitHub PR, Base Branch, Compare Branch, Code Review, @mention, Request Changes, Approve, Merge Button, git push origin, Branch Protection, Discussion Thread, PR Lifecycle, Commit History
