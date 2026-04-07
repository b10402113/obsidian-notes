# 自動化是 DevOps 的基石

## 📝 課程概述

本單元深入探討自動化在 DevOps 中的核心地位，以及如何為 Docker 專案選擇正確的自動化工具與工作流。講師分享了產業趨勢觀察，並提出「Top 10」應該自動化的項目清單。

---

## 核心觀念與實作解析

### 自動化平台的整合趨勢

市場上自動化工具數以百計，但產業正在經歷 **整合（Consolidation）**：

- 勝出的平台是那些 **與程式碼存放位置深度整合** 的工具
- GitHub、GitLab、Bitbucket 成為主流選擇
- **Plugin 生態系** 是關鍵競爭力

#### 為什麼 Plugin 系統很重要？

理論上，我們可以用 Bash 腳本完成所有自動化。但現代自動化平台的價值在於：

- **社群貢獻的 Plugin** 將複雜任務封裝成簡單的設定
- 「拖拉式」層級的易用性
- 開源標準讓 Plugin 可以跨專案複用

> **講師觀點**：如果一個自動化工具沒有龐大的 Plugin 生態系，你會花大量時間在寫 Bash 腳本上，這應該盡量避免。

---

### Docker 專案的自動化 Top 10

講師提出了 Docker 專案應該自動化的核心項目：

| 階段 | 自動化項目 |
| --- | --- |
| **PR 階段** | Lint、Build、Test、Security Scan |
| **合併階段** | Push Image、Deploy |

#### 基礎工作流

```
PR Push → Build Image → Test Image → (反覆修改) → Merge → Push to Registry → Deploy
```

這是一個線性流程：每次 commit 都會觸發 Image 建置與測試。

---

### Linting：不能再被忽視

講師特別強調 **Linting** 的重要性：

#### 推薦工具

- **Super Linter** — GitHub 官方維護，支援多種語言
- **Mega Linter** — 社群維護，功能更豐富

#### 為什麼一定要做 Linting？

> **講師立場**：Linting 不應該是選配的。它不複雜，不需要大量工作，但長期效益巨大。

- 所有語言、所有檔案類型都應該有一致的 Linting 標準
- 在 PR 階段就執行，讓開發者在合併前就知道需要修正什麼
- 講師在所有專案中都達成 100% Linting 覆蓋率

---

### 假設：PR-based 工作流

本系列課程假設你使用 **PR-based 工作流**：

1. 從 main 分支建立新分支
2. 進行修改並建立 PR
3. 自動化流程被觸發（Lint、Test、Build、Scan）
4. 所有檢查通過後合併
5. 合併後執行部署相關自動化

> 如果你的團隊尚未採用 PR 工作流，講師建議盡快導入，這是現代 DevOps 的基礎。

---

## 💡 重點摘要

- **自動化平台正在整合到程式碼倉庫中，GitHub Actions 因為龐大的 Plugin 生態系而成為首選。**
- **Linting 是必備的自動化項目，推薦使用 Super Linter 或 Mega Linter，應在 PR 階段執行。**
- **PR-based 工作流是現代 DevOps 的標準，自動化在 PR 建立與合併兩個時機觸發。**
- **Docker 專案的核心自動化包括：Lint、Build、Test、Scan、Push、Deploy。**

---

## 🔑 關鍵字

GitHub Actions, Super Linter, Linting, Plugin, PR Workflow
