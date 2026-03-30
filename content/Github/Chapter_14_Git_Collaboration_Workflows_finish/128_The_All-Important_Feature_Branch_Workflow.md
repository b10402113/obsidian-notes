# Feature Branch Workflow——多人協作的正確起點

## 📝 課程概述

本單元介紹 **Feature Branch Workflow**——所有成員**永遠不在 main/master 上直接作業**，而是為每個功能或任務建立獨立的 branch，開發完成後再 merge 回 main。這種方式從根本上杜絕了 Centralized Workflow 的三大致命問題，讓團隊得以安全地實驗、協作，並保持 main branch 的穩定。

## 核心觀念與實作解析

### 核心原則：Nobody works on main

**Feature Branch Workflow 的唯一鐵律：沒有人直接在 main/master 上 commit。所有開發行為都發生在 feature branch 上。**

> 這就像：「家裡的客廳是公共區域，你不能在客廳裡面組裝家具。你必須到自己的工作間（feature branch）去做，完成後才能把成品帶進客廳。」

### 與 Centralized Workflow 的對比

| 情境 | Centralized Workflow | Feature Branch Workflow |
|------|--------------------|----------------------|
| 新功能開發 | 直接在 main 上 commit | 建立 feature branch 來做 |
| 分享不完整程式碼 | 只能推送到 main，破壞所有人 | 推送到自己的 feature branch，main 不受影響 |
| 請求同事協助 | 必須先 merge main 再 push，有 Bug 的程式碼直接進入 main | 直接在 feature branch 上協作，main 乾淨 |
| Merge 回 main | 隨時可以 merge | Feature 完成、通過審查後再 merge |

### 多人協作的實際流程

```
David 想要一個 Dark Theme 功能：
1. git switch -c add-dark-theme   # 建立新 branch
2. 開發 30%... 功能還沒完成
3. 他想把程式碼分享給 Pamela 看看
4. 他直接 git push origin add-dark-theme  ✓ 成功
5. Pamela: git fetch → git switch add-dark-theme  # 直接在 feature branch 上 review 並協作
6. Pamela 在同一個 feature branch 上追加 commit
7. 最終 feature 完成，merge 到 main
```

** Pamela 不需要停止自己的工作**（假設她同時在 `animated-scroll` branch 上作業），她只需要 fetch David 的 branch、checkout、貢獻程式碼，main branch 從頭到尾都沒有被碰過。

### Feature Branch 的命名慣例

分支名稱沒有強制標準，但大型組織通常遵循一些命名慣例：

- `feat/navbar` —— 新功能
- `bugfix/login-error` —— Bug 修復
- `refactor/auth-module` —— 重構
- `chore/update-dependencies` —— 日常維護

團隊內部應提前約定命名規則，以利在 branch list 中快速識別各個分支的用途。

### Feature Branch 的合併方式

當一個 feature 完成後，合併回 main 的方式有幾種：

1. **Free merge**：任何人想 merge 就直接 merge，不需討論（小型 hobby 專案）
2. **Discussion then merge**：merge 前先跟團隊討論（中型團隊）
3. **Pull Request**：透過 PR 提交讓人 review，審查通過後才 merge（這是下一個單元的主題）

> **重要認知**：Feature Branch Workflow 和 Pull Request 是兩個獨立的觀念。Feature Branch Workflow 是「在哪個 branch 上開發」的策略；Pull Request 是「如何把 branch 合併進 main」的審查流程。兩者經常一起使用，但也可以分開。

### 為什麼 main branch 是「神聖」的？

main branch 代表的是**專案隨時可以發布的穩定狀態**。它應該：

- 永遠包含經過測試、不影響其他成員的程式碼
- 不包含任何「做到一半」的功能
- 是團隊共享的 truth source（單一真相來源）

當所有成員都在 feature branch 上作業，main 就成為了那個「永遠可用」的 stable branch。**一旦破壞 main 的穩定性，整個團隊的工作節奏都會被打亂。**

## 💡 重點摘要

- **Feature Branch Workflow 的核心精神**：永遠不在 main/master 上直接開發，所有工作都在 feature branch 上完成。
- **Main branch 是 stable 的保證**：它代表專案隨時可以交付的狀態，團隊成員可以隨時從 main 分支出來、合併回去，不需擔心被別人的實驗性程式碼影響。
- **分享不完整程式碼不需要破壞 main**：只需要把 feature branch push 到 GitHub，團隊成員 fetch 之後就可以在同一条 branch 上協作。
- **分支是「功能導向」的**：一個 branch = 一個功能或一個 Bug 修復，這讓 branch 的生命週期非常清晰。
- **Feature Branch Workflow + Pull Request** 是現代軟體開發的黃金組合，也是下一個單元的主題。

## 關鍵字

Feature Branch Workflow, Main Branch, Stable Branch, git switch -c, Branch Naming Convention, feat/, bugfix/, Branch Lifecycle, Stable Main, git push origin, Fetch Remote Branch, Collaboration, Code Review, Feature Isolation
