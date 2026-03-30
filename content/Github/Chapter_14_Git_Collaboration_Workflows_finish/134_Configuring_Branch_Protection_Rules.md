# Branch Protection Rules——保護你的 Main Branch

## 📝 課程概述

本單元介紹 GitHub 的 Branch Protection Rules（分支保護規則）。這個設定讓你可以**強制要求 Pull Request 才能合併進 main branch**，並要求至少一個（或多個）審查者 approve 後才能按下 Merge 按鈕。這是防止團隊成員意外（或故意）破壞 main branch 的最後一道防線。

## 核心觀念與實作解析

### 為什麼需要 Branch Protection？

即使團隊有共識「不在 main 直接 push」，**人為錯誤永遠可能發生**——有人可能在壓力下直接 commit 到 main，或是不小心選錯 branch。這時 Branch Protection 就是一道安全閥，**直接阻擋未經審查的變更**。

### 設定 Branch Protection Rules

在 GitHub repository → **Settings** → **Branches** → **Add rule**。

**Branch name pattern（分支名稱模式）**：

- 如果只要保護 `main`：直接輸入 `main`
- 如果要保護所有 `main` 和 `master`：可以使用萬用字元 `main*`
- 大型專案可能用 pattern 如 `feat/*`、`release/*`，對不同類型的分支設定不同規則

### Require Pull Request Reviews Before Merging

這是最關鍵的設定：**合併進 protected branch 之前，必須透過 PR 流程，且至少要有 N 個 reviewer approve。**

```
Require pull request reviews before merging
  ├── Required number of reviewers: 1 (至少需要 1 人批准)
  ├── Dismiss stale reviews (新的 push 是否自動撤銷舊的審查)
  └── Require review from Code Owners (特定目錄的變更需要該目錄 owner 批准)
```

設定後，如果有人嘗試直接 commit 到 protected branch：

```
! [remote rejected] main -> main (refusing to allow an integration to delete the main branch)
remote: error: GH006: Protected branch hook declined to update main
```

### 實際效果：從 Stev ie 的視角看保護後的差異

**設定保護規則之前：**

- Stevie 可以直接在 GitHub 網頁上 commit 到 main
- Stevie 的 PR 可以自己 merge（沒有人把關）

**設定保護規則之後：**

- Stevie **無法直接在 main 上 commit**，GitHub 會阻止
- Stevie 的 PR 頁面：Merge 按鈕被禁用，顯示「At least 1 approving review is required」
- **Colt 必須在 PR 頁面點擊 "Review" → "Approve"**，Merge 按鈕才會啟用

### Approve 流程的實作

審查者（Colt）在 PR 頁面可以：

1. **Comment**：純留言，不影響 Merge 按鈕
2. **Request Changes**：要求開發者修改，會阻擋 Merge，直到再次 review
3. **Approve**：表示同意，Colt 的 approve 讓 Merge 按鈕啟用

```
開發者 Stevie → 開 PR → 審查者 Colt → Comment/Request Changes/Approve
                                            ↓
                                     Colt 的 "Approve" = 同意合併
```

### 其他常見的 Branch Protection 選項

| 設定選項 | 意義 |
|---------|------|
| **Require status checks to pass before merging** | 要求 CI/CD（如 GitHub Actions）測試全部通過才能 merge |
| **Require branches to be up to date before merging** | 合併前確保 feature branch 基於最新的 main |
| **Do not allow bypassing of above rules by administrators** | 即使是 repository owner 也必須遵守保護規則 |
| **Allow force pushes** | 是否允許 force push（通常在 protected branch 上應關閉） |
| **Allow branch deletion** | 是否允許刪除該分支 |

### Branch Protection 在開源專案中的應用

像 React 這種大型開源專案，Branch Protection 通常設定得非常嚴格：

- **至少 1-2 個 maintainer approve** 才能 merge
- **所有 CI 測試必須通過**
- **Owner 無法繞過規則**（防止權力集中）

這確保了即使是 React core team 的成員，也必須經過完整的審查流程才能變更 main。

## 💡 重點摘要

- **Branch Protection Rules 是 GitHub 的存取控制機制**：它讓「只有通過 PR 並獲得批准才能合併進 main」成為技術上的強制要求，而非僅靠團隊成員的自律。
- **Require PR Reviews** 是最核心的設定——它把 Code Review 從「建議」變成「必須」。
- **設定保護規則後，direct commit 會被 GitHub 阻止**：即使你是 repository owner，如果規則設定嚴格，也無法直接推送 protected branch。
- **Approve 數量可以客製化**：1 人、2 人、根據目錄設定不同的要求，適用於各種規模的團隊。
- **保護 main branch 對開源專案尤其重要**：讓 thousands of contributors 都能透過 PR 貢獻，但只有 maintainer 可以決定什麼進入 main。

## 關鍵字

Branch Protection Rules, Require PR Reviews, Approve Review, Merge Button, Protected Branch, Pull Request, Repository Settings, CI Status Checks, Bypass Rules, Force Push, Code Owner Review, Dismiss Stale Reviews, Collaboration Workflow, Repository Administrator
