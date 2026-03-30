# Git 協作工作流程導論

## 📝 課程概述

本單元預覽整個協作工作流程章節的學習地圖。我們會依序探討：Centralized Workflow 的缺陷、Feature Branch Workflow 的核心概念、Pull Request 的機制與實作、Branch Protection Rules 的設定方法，以及最後的 Fork and Clone Workflow——這是開源專案協作的標準模式。理解這些工作流程，會讓你在任何規模的團隊中都能游刃有餘。

## 核心觀念與實作解析

### 為什麼工作流程這麼重要？

在 Git 的操作命令之外，**「什麼時候、在哪個分支、用什麼方式」提交與合併程式碼**，才是真正決定團隊協作效率的關鍵。使用錯誤的工作流程，會讓一個功能還沒完成就弄壞整個 main branch；使用正確的工作流程，則可以安全地實驗、協作、審查，最後才把通過檢驗的程式碼納入正式版本。

### 四大協作工作流程的地圖

| 工作流程 | 核心概念 | 適用情境 |
|---------|---------|---------|
| **Centralized Workflow** | 全體成員在單一 branch（main/master）上工作 | 小型 hobby 專案，**不建議團隊使用** |
| **Feature Branch Workflow** | 所有人離開 main branch，在各自的 feature branch 上作業 | 小型到中型團隊協作 |
| **Pull Request 流程** | 透過 PR 提交、審查、討論後再合併 | 中型到大型團隊，有程式碼審查需求 |
| **Fork and Clone Workflow** | 每個Contributor 擁有自己的 fork，用 PR 與原專案同步 | 大型開源專案（React、VS Code 等） |

### 章節中每個環節的價值定位

**PR（Pull Request）是最重要的環節**——如果你計畫與他人合作、參與開源專案，或在任何有程式碼審查制度的公司工作，PR 都是你幾乎每天都會接觸到的工具。

Branch Protection Rules 屬於「重要但非核心」的設定——知道它的存在很重要，但學會 PR 操作流程之後再學這個也不遲。

Fork and Clone Workflow 是開源貢獻的必備知識。只要你想對 React、TensorFlow、VS Code 這類擁有數千名貢獻者的專案提出貢獻，你就必須理解這個流程，因為**這些專案不可能讓每個人都成為直接 collaborator**。

## 💡 重點摘要

- 工作流程決定了團隊如何組織開發節奏，選擇錯誤的工作流程會讓簡單的協作變成一場災難。
- **Feature Branch Workflow** 是大多數團隊的起點，能有效隔離實驗性質的開發與 stable 的 main branch。
- **Pull Request** 是現代軟體開發中程式碼審查（Code Review）的事實標準，GitHub、GitLab、Bitbucket 都以此為核心功能。
- **Fork and Clone Workflow** 讓沒有直接 push 權限的人也能參與開源專案，這是開源生態系的協作基石。
- 理解工作流程比學新命令更重要——本章節不教你新 Git 命令，而是教你「什麼時候該用哪個命令」。

## 關鍵字

Collaboration Workflow, Centralized Workflow, Feature Branch Workflow, Pull Request, Branch Protection Rules, Fork and Clone Workflow, Code Review, Open Source, Remote Repository, Collaborator, Main Branch, Merge, SSH Key
