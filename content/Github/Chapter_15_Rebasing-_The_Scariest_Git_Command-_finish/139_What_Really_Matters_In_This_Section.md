# What Really Matters In This Section — Rebase 核心觀念導讀

## 📝 課程概述

本單元只聚焦一個指令：`git rebase`。這是一個長期存在於 Git 社群、但充滿爭議的命令——有人天天使用，有人刻意迴避。我們將在這個章節裡弄清楚：**什麼是 rebase？它與 merge 的差異在哪裡？什麼時機適合用？什麼時機絕對不能碰？** 了解這些，你就能在需要的時候自信地使用它。

## 核心觀念與實作解析

### 為什麼 rebase 讓人又愛又怕？

 instructor 在影片中分享了自己與 rebase 的「愛恨情仇」。一開始接觸 Git 時，指導他的人告誡他：「盡量別用 rebase，它會把事情搞砸，尤其對初學者不友善。」 instructor 因此避開 rebase 多年，靠 `git merge` 依然過得很好。後來在授課時被學生當場問倒，才認真研究，現在反而天天都在用。

這個轉變的關鍵在於：**理解 rebase 的本質——它並不危險，危險的是在錯誤的時機使用它。** 一旦掌握「什麼時候絕對不能 rebase」，它就是一個強大且能讓團隊協作更順暢的工具。

### Git 社群對 rebase 的兩極態度

> Rebase 在社群中是一個充滿分歧的話題——有些人每家公司都要求他們用 rebase，有些則完全避開。

這背後沒有對錯，而是取決於：
- **團隊規模**：單人專案、兩三人小團隊 vs. 大型多人協作
- **組織的 Git workflow**：有些公司要求乾淨的 commit 線性歷史，有些則不在意
- **個人偏好與經驗**

### rebase 的兩大用途

 instructor 將 rebase 的應用分成兩種情境：

1. **替代 `git merge`**（本課程重點）：用 rebase 而非 merge 來整合不同分支的變更，藉此產生乾淨的線性歷史
2. **整理自己的 commit 歷史**：在即將合併回主分支前，用互動式 rebase（interactive rebase）壓縮、刪除或修改 commit 訊息

本單元先聚焦第一種用法。

### 為什麼 instructor 說 rebase「不是必學命令」？

 instructor 坦言，他用了很長一段時間 Git，幾乎沒怎麼用到 rebase，照樣完成所有工作。`git merge` 能做到幾乎所有 rebase 能做到的事，只不過 commit 歷史會有不同的樣貌。**rebase 不是「沒有它就會死」的指令，但它值得學，因為越來越多公司的工程團隊要求或期待你熟悉這套 workflow。**

## 💡 重點摘要

- `git rebase` 和 `git merge` 都用來整合分支變更，但兩者產生的歷史結構完全不同
- rebase 在 Git 社群有爭議——了解背後原因，比急著使用更重要
- rebase 不是新手必修，但絕對是值得知道的技術，因為業界確實有團隊在用
- rebase 有兩種主流用途：替代 merge，以及清理 commit 歷史
- **Golden Rule（黃金守則）**：千萬不要 rebase 已經分享給他人的 commit——這是所有災難的根源

## 關鍵字

`git rebase`, `git merge`, Branch, Feature Branch, Commit History, Linear History, Rewriting History, Golden Rule of Rebasing
