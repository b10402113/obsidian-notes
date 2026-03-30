# Interactive Rebase 入門：Rewrite History 的核心觀念

## 📝 課程概述

本單元介紹 Git Rebase 的第二種應用場景——**Interactive Rebase（互動式變基）**。不同於前幾章將 Rebase 作為 Merge 的替代方案，本章我們學會用它來「**重寫歷史**」：在分享程式碼之前，先將本地的 commit 記錄整理得清晰、簡潔。這是許多開發者在正式開 PR 或分享分支前，會實際執行的 Workflow 步驟。

## 核心觀念與實作解析

### 什麼是 Interactive Rebase？

**Interactive Rebase** 的核心精神是：允許你逐一檢視並修改某段範圍內的每一個 commit。你可以：

- **Reword**：修改 commit message
- **Edit**：修改 commit 的實際內容
- **Fixup / Squash**：合併多個相關的 commit
- **Drop**：直接刪除某個 commit
- **Reorder**：調整 commit 的順序

> 這些操作全部發生在**本地**，在程式碼尚未與他人共享之前執行，這一點至關重要。

### 為什麼 Rewriting History 需要謹慎？

當我們修改一個 commit 的內容或訊息時，Git 會產生一個**新的 commit hash**。這是因為每個 commit 的 hash 都包含了它的父 commit 資訊——一旦父 commit 變了，hash 就必然不同。

所以當你 Reword 了某個 commit，所有**在此之後的 commit** 都會因為 parent hash 的變化而被重新計算，全部變成新的 commit。這在本地無害，但如果團隊成員已經基於這些 commit 展開工作，這樣的改動會讓他們的歷史與你的產生分歧，造成混亂。

這就是為什麼：
> **千萬不要在已經推送到 remote 或其他人已經取得的 commit 上執行 interactive rebase。**

### 適合的使用時機

- 你已經在 local 的 feature branch 上開了許多半成品 commit（e.g., "fix typo", "whoops forgot X"）
- 在正式開 PR 之前，想要把這些瑣碎的 commit 整合成有意義的歷史敘述
- 符合專案的 commit message 規範（例如：要求使用 imperative mood）

### 與一般 Rebase 的語法差異

```bash
# 一般 Rebase：將 branch A 變基到 branch B
git rebase branchA  # 指定要 rebase 到哪個 branch

# Interactive Rebase：原地重建最近 N 個 commit
git rebase -i HEAD~N  # 不指定目標 branch，只指定要重建的範圍
```

`-i` 是 `--interactive` 的縮寫，執行後會開啟編輯器，讓你對範圍內的每個 commit 下達指令。

### Interactive Rebase 的編輯器邏輯

執行 `git rebase -i HEAD~N` 後，編輯器會以** oldest → newest** 的順序（與 `git log` 相反）列出 N 個 commit，格式如下：

```
pick abc1234 initial commit
pick def5678 add project files
pick ghi9012 add bootstrap
# ...
```

Git 會根據你修改後的指令，**由上而下依序執行**每個 commit 的處理方式——這也是為什麼順序很重要。

## 💡 重點摘要

- Interactive Rebase 是本地重寫 Git 歷史的強大工具，適合在分享程式碼前整理 commit
- 執行 `git rebase -i HEAD~N`，Git 會開啟編輯器讓你對 N 個 commit 逐一下達指令
- 編輯器中 commit 順序是 oldest → newest，與 `git log` 相反
- 改變任何一個 commit 都會導致其後所有 commit 產生新的 hash
- **千萬不要對已經與他人共享的 commit 執行 interactive rebase**

## 關鍵字

Interactive Rebase, `git rebase -i`, rewrite history, reword, fixup, squash, drop, commit hash, imperative mood, feature branch cleanup
