# Re-Attaching Our Detached HEAD! — 穿越歷史與重新附著

## 📝 課程概述

本節深入探討 Detached HEAD 狀態的意義，以及如何在這個狀態下進行操作。當你使用 `git checkout <commit-hash>` 而非分支名稱時，Git 會讓 HEAD 直接指向一個具體的 Commit，而非分支引用。這個狀態讓你得以「時間旅行」觀看歷史，但若要繼續在上面工作，你需要建立新分支來重新附著 HEAD。

## 核心觀念與實作解析

### HEAD 的正常行為

在正常情況下，HEAD 的指向結構是：

```
HEAD → branch reference（如 master）→ Commit
```

也就是說，HEAD 永遠**指向一個分支**，分支再指向 Commit。當你 `git commit` 時，分支 Pointer 向前移動，但 HEAD 仍然指向同一個分支。

### Detached HEAD 的結構

當你執行 `git checkout <commit-hash>` 時：

```
HEAD → Commit（直接指向，脫離了分支）
```

這就是 Detached（脫離）的含義——**HEAD 不再綁定到任何分支**。

### Detached HEAD 的意義：純閱讀，不影響歷史

在 Detached HEAD 狀態下，Git 會清楚告訴你：

> "You are in 'detached HEAD' state. You can look around, make experimental changes and commit them, and you can discard any commits you make in this state without impacting any branches by switching back to a branch."

翻譯成白話：你可以四處看看、做實驗性修改、嘗試 Commit——但**這些 Commit 沒有分支可以依靠**，如果你切回分支而不建立新分支，這些實驗 Commit 會成為「孤兒 Commit」（最終被 Git GC 回收）。

### 三種離開 Detached HEAD 的方式

**方式一：直接切回分支（不保留實驗 Commit）**
```bash
git checkout master
# 或
git switch master
```
所有在 Detached HEAD 期間的 Commit 都會消失（因為沒有分支引用它們）。

**方式二：使用 `git switch -`（快速回到上一個分支）**
```bash
git switch -
```
`git switch -` 是 `git checkout -` 的現代化寫法，意為「切換到上一次所在的分支」。

**方式三：在 Detached HEAD 狀態建立新分支（最常見的實用做法）**
```bash
# 在 Detached HEAD 狀態下：
git switch -c <new-branch-name>
# 或
git checkout -b <new-branch-name>
```
這樣 HEAD 會重新附著到一個新的分支上，你在這個歷史點的所有工作都可以被保存下來。

### `HEAD~n` 語法：以 HEAD 為基準的相對引用

除了使用 Commit Hash 之外，你還可以用 `HEAD~n` 語法來引用距離 HEAD n 步的祖先 Commit：

| 語法 | 含義 |
|------|------|
| `HEAD~1` | HEAD 的父 Commit（上一個）|
| `HEAD~2` | HEAD 的祖父 Commit（上兩個）|
| `HEAD~n` | HEAD 往前 n 步的 Commit |

```bash
# 回到上一個 Commit（進入 Detached HEAD）
git checkout HEAD~1

# 回到上三個 Commit
git checkout HEAD~3
```

> **實用價值**：當你只是需要快速回到「上一次 Commit」或「上兩次 Commit」時，不需要複製 Hash，直接用 `HEAD~n` 語法即可。

### 實作：時間旅行的完整流程

**情境**：你發現某個 Commit 之後的功能有問題，想回到那個時間點重新開始。

1. `git log --oneline` 查看歷史，找到目標 Commit Hash（如 `4f1b...`）
2. `git checkout 4f1b...` → 進入 Detached HEAD，檔案內容回到那個時間點
3. 在這裡，你可以：
   - **純觀看**：直接 `git switch master` 回來
   - **分支發展**：在這個時間點上建立新分支 `git switch -c new-feature`
   - **繼續前進**：在這個時間點上開始新的工作線

> **關鍵理解**：Detached HEAD 是一種「安全的历史檢視模式」。Git 允許你自由地穿梭在歷史中，而不必擔心會不小心弄亂現有的分支結構。

## 💡 重點摘要

- Detached HEAD 的成因：`git checkout <commit-hash>` 直接讓 HEAD 指向 Commit，而非分支
- Detached HEAD 不是錯誤，是一種允許你「觀看歷史」的隔離狀態
- 離開 Detached HEAD：`git switch <branch>` 回到分支，或 `git switch -` 回到上個分支
- 在 Detached HEAD 狀態建立分支：`git switch -c <branch-name>`，可以保留你在歷史時間點上的所有工作
- `HEAD~n` 語法是以 HEAD 為基準的相對引用：`HEAD~1` 是上一個 Commit，`HEAD~2` 是上兩個，以此類推
- Detached HEAD 期間如果做了 Commit 但沒有建立分支，切回分支後這些 Commit 會成為孤兒（最終被回收）

## 關鍵字

detached HEAD, git checkout, git switch, HEAD, HEAD~1, HEAD~2, commit hash, branch reference, git log --oneline, time travel, reattach
