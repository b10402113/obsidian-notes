# Centralized Workflow 示範——單一分支的三大致命問題

## 📝 課程概述

本單元透過實際操作演示 Centralized Workflow（全員在同一分支工作）的災難性後果。我們會看到當多個開發者同時在 `main` 或 `master` 分支上 commit 時，push 被拒絕、必須強制 merge、以及最糟糕的情況——**不完整的程式碼被迫進入 main branch、破壞整個團隊的程式碼庫**。

## 核心觀念與實作解析

### Centralized Workflow 的定義

**Centralized Workflow = 全體成員在同一個 branch（通常為 main/master）上作業。** 所有 commit 都在同一條 line 上，沒有 feature branch。

這是看起來最簡單的協作模式，但正因為太過簡單，反而埋下諸多問題。

### 問題一：每次 push 前都必須先 merge

實際上演練的場景是這樣的：

```
 Colt 和 Stevie 共同使用同一個 repo
 1. Colt 做了兩個 commit（新增 index.html、使用 Bootstrap）
 2. Colt git push origin main  ✓ 成功
 3. Stevie 同時也在 index.html 上作業（新增 Navbar）
 4. Stevie 嘗試 git push origin main  → ✗ 被拒絕
```

Git 拒絕的原因是：**你的 branch 目前落後於 remote。** 你必須先 `git pull` 把別人的新 commit 拉下來，與你的 commit merge 成一個 merge commit，然後才能 push。

```bash
# Stevie 被拒絕後的補救流程
$ git pull origin main
# 可能產生 merge conflict
# 解決衝突 → add → commit → 終於可以 push
$ git push origin main
```

> **為什麼這很痛苦？** 因為這不是「完成一個功能之後才合併」，而是「每次想要分享任何進度時」都必須先 merge。如果團隊有 5 個人，每個人每天都要重複這個流程，光 merge 就佔據大量時間。

### 問題二：無法實驗

Centralized Workflow 最大的心理障礙是：**main/master 是唯一的 stable branch，但也是你唯一能 commit 的地方。**

如果要測試一個大改版、或 try 一些很激進的重構，你沒有地方可以「安全地破壞程式碼」——因為無論你在哪個 branch commit，最後要分享給團隊，就必須把那些 commit 送進 main branch，而團隊每個人都會被影響。

### 問題三：協作必須「污染」main branch

這是災難的核心場景：

> Stevie 寫了一個 Navbar，但有 Bug 無法正常展開。他需要 Colt 的幫助，於是想把程式碼分享給 Colt 看。但 Centralized Workflow 下，他要分享就必須直接 commit 到 main branch——**main branch 因此出現了未完成、有 Bug 的程式碼，所有團隊成員都受影響。**

```
David 在 main branch 上 commit 了不完整、有 Bug 的程式碼
  → 推送到 GitHub
  → 所有人 git pull 後，程式碼庫的 main branch 就「壞掉」了
  → 只因為他想要尋求同事的意見
```

**在真實工作情境中，這會導致整個團隊隔天上班時，pull 下來的程式碼是無法運作的版本。**

### 團隊規模放大後的災難

想象一個 10 人團隊：

- 每個人每天下班前都要先 pull所有人的 commit
- merge 自己零碎的 commit（有些甚至只是「做到一半」）
- 解決大量潛在的 merge conflict
- 才能 push 上去

這完全違背了版本控制應該帶來的「清晰、可追溯」體驗。**工作流程的混亂會直接轉化為團隊的壓力與錯誤。**

## 💡 重點摘要

- **Centralized Workflow 的本質缺陷**：沒有 feature branch，所有人共享一個隨時可能被破壞的 stable branch。
- **每次分享進度都需先 merge**：這在小團隊還能忍受，團隊一大就變成每個人的噩夢。
- **無法隔離實驗性開發**：想要 try something radical 只能直接改 main branch，後果由全團隊承擔。
- **協作需求會迫使成員提交不完整程式碼**：這是災難的起點——main branch 不再 stable。
- **結論**：Centralized Workflow 只適合單人或極小型的 hobby 專案，任何有協作需求的團隊都應該採用 Feature Branch Workflow。

## 關鍵字

Centralized Workflow, Shared Main Branch, Push Rejected, git pull before push, Merge Conflict, Broken Main Branch, Collaboration, Experiment, Unfinished Code, Remote Tracking
