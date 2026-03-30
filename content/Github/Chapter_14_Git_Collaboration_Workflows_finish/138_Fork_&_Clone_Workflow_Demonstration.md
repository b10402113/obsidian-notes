# Fork and Clone Workflow——完整的開源協作流程

## 📝 課程概述

本單元將 Forking 與完整的協作流程整合在一起，展示 **Fork and Clone Workflow** 的全貌。這套工作流程讓沒有原專案 push 權限的人，也能夠安全地進行本地開發、透過 fork 分享變更，並透過 Pull Request 向原專案提交貢獻。這是參與 React、TensorFlow、VS Code 等大型開源專案的標準方式。

## 核心觀念與實作解析

### Fork and Clone Workflow 的完整步驟

```
1. Fork 專案（GitHub 頁面操作）
2. Clone 自己的 fork 到本地
3. 新增 upstream remote（指向原專案）
4. 在本地開發（最好建立 feature branch）
5. Push 到自己的 fork
6. 發起 Pull Request（從 fork 到 upstream 原專案）
7. 等待 maintainer 審查
8. （如果 PR 被接受）同步 upstream 的最新變更
```

### 完整實作循環

**步驟 1-3：設定好本地的 remote 環境**

```bash
# Fork 在 GitHub 上完成

# Clone 你的 fork
$ git clone https://github.com/YOUR_USERNAME/REPO_NAME.git

# 加入 upstream remote
$ git remote add upstream https://github.com/ORIGINAL_OWNER/REPO_NAME.git
```

**步驟 4：在本地建立 feature branch 開發**

```bash
$ git switch -c feat/improve-something

# 進行開發與 commit
$ git add .
$ git commit -m "add improvement"
```

> **為什麼用 feature branch？** 因為你的 PR 應該只包含一個明確的功能或修復。直接用 main branch PR 會讓 maintainer 難以審查，而且當你需要與 upstream 同步時，會造成更多 conflict。

**步驟 5：Push 到你的 fork**

```bash
$ git push origin feat/improve-something
```

**步驟 6：在 GitHub 上發起 PR**

在 GitHub 的 **Your Fork** 頁面 → 切換到 `feat/improve-something` branch → 點擊 **Compare & pull request**。

GitHub 會聰明地自動選擇：
- **base repository**：原專案（原 upstream）
- **base**：目標 branch（如 `main`）
- **head repository**：你的 fork

> 這與 collaborator 的 PR 最大的不同在於：base 是原專案，而 collaborator 的 PR 是從一個 branch 到同一個 repository 的另一個 branch。

**步驟 7：Maintainer 審查並合併**

原專案的 maintainer 收到 PR 後，會：
1. Review 程式碼
2. 留言討論或要求修改
3. 如果滿意，點擊 **Merge**

**步驟 8：同步 upstream 的最新變更**

```bash
$ git switch main
$ git fetch upstream
$ git merge upstream/main   # 把原專案的新變更同步到你的 fork
$ git push origin main     # 同步到 GitHub 上的 fork
```

### Fork 的兩個 Remote 視角

```
┌─────────────────────────────────────────────┐
│  upstream (原專案 - 你沒有 push 權限)         │
│  https://github.com/ORIGINAL/REPO           │
│                                             │
│  origin (你的 fork - 你有完整權限)             │
│  https://github.com/YOUR_USERNAME/REPO      │
└─────────────────────────────────────────────┘
        ↑ PR (From your fork)
        │
   Pull Request destination
```

### 為什麼 Feature Branch 很重要？

在 collaborator 的情境中，PR 從 feature branch → main。

在 Fork and Clone 的情境中，PR 從 **fork 的 feature branch** → **upstream 的 main**。

如果你的 fork 在 main 上直接做 commit，當 upstream 有新變更時，你的 main 與 upstream/main 會有很大的分歧，merge 會產生大量 conflict。用 feature branch 可以隔離你的變更，conflict 只發生在 PR 層次，比較容易管理。

### Fork and Clone vs. 直接 Clone 的根本差異

| 情境 | Fork and Clone | 直接 Clone（作為 Collaborator） |
|------|--------------|-----------------------------|
| Push 目的地 | 自己的 fork（origin） | 原專案（origin） |
| 需要 upstream remote？ | **需要**（同步原專案變更） | 不需要（你直接操作原專案） |
| PR 的 base | upstream 原專案 | 同一個 repository 的 main |
| 是否需要 Fork？ | **需要** | 不需要 |

### 真實開源專案的 Fork and Clone 案例

以 React 為例：

```
Ari's fork of React (https://github.com/AriPerkkio/react)
  → PR → Facebook/react (原專案 upstream)

React PR 頁面明確標示：
"ARI PERKKIO wants to merge 1 commit into facebook:master from AriPerkkio:patch-1"
```

這個 PR 從 Ari 的 fork (`AriPerkkio/react`) 的 `patch-1` branch，合併進 Facebook 的原專案 (`facebook/react`) 的 `master` branch。

### 為什麼你不應該直接推送到原專案

作為 contributor，你對原專案**沒有任何 push 權限**。嘗試這樣做會得到：

```
! [remote rejected] master -> master (permission denied)
remote: fatal: Permission to facebook/react.git denied to AriPerkkio
```

**Fork 就是解決這個問題的設計**：你不能寫入原專案，但你可以在自己的 fork 上做任何操作，然後透過 PR 請求原專案 pull 你的變更。

## 💡 重點摘要

- **Fork and Clone Workflow 是開源貢獻的標準流程**：你不會獲得原專案的 push 權限，但你會得到一個完整的個人 fork 來進行開發。
- **`origin` = 你的 fork，`upstream` = 原專案**——這個命名約定讓整個團隊的溝通更清晰。
- **每個功能或修復都應該在獨立的 feature branch 上作業**，這讓 PR 的範圍明確，且未來同步 upstream 時 conflict 最小。
- **PR 從 fork 的 feature branch → upstream 的 main**——這是 Fork and Clone 與 collaborator PR 的根本差異。
- **定期同步 upstream**：長期參與一個開源專案，你需要經常 `git fetch upstream` 並 merge 最新變更到你的 local main，保持 fork 不落後太多。

## 關鍵字

Fork and Clone Workflow, Origin, Upstream, git remote add upstream, Pull Request to Upstream, Open Source Contribution, Feature Branch in Fork, Sync Upstream, facebook/react, Contributor, Maintainer, Collaborator vs Contributor, Permission Denied, Patch Branch, Merge Upstream
