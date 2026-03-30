# Git Reset & Git Revert — 撤銷 Commit 的兩種思維

## 📝 課程概述

本節介紹 `git reset` 和 `git revert` 這兩個用來「撤銷 Commit」的指令。兩者名稱相似、功能相似（都會讓專案回到過去的狀態），但**底層機制截然不同**。`git reset` 透過**刪除 Commit 來改寫歷史**；`git revert` 透過**新增一個反向 Commit 來保留歷史**。這個差異在協作場景中極為關鍵，是本節最重要的觀念。

## 核心觀念與實作解析

### `git reset`：歷史改寫

```bash
git reset <commit-hash>
```

`git reset` 的運作方式是：
1. 將 Branch Pointer 向**後移動**到指定的 Commit
2. 指定範圍內的所有 Commit **被刪除**
3. 根據模式，Working Directory 的內容會有不同的命運

#### 普通 `git reset`（預設模式）

```bash
git reset abc1234
```

**只移動 Branch Pointer，不改變 Working Directory**。被刪除的 Commit 中的**變更仍然留在 Working Directory 中**，你可以自由決定如何處理它們。

**實作情境**：
- 在 `master` 上連續做了兩次錯誤的 Commit：「mistake commit」和「another bad commit」
- `git reset 4th-commit-hash` → 這兩個 Commit 被刪除，但檔案內容仍然在 Working Directory
- 你可以選擇：建立新分支把這些變更移過去，或直接 discard

#### `--hard` 模式

```bash
git reset --hard abc1234
```

**Branch Pointer 向後移動，且 Working Directory 也被「倒帶」到那個 Commit 的狀態**。所有被刪除 Commit 的變更**全部消失，無法恢復**。

**實作情境**：
- 執行 `git reset --hard 3rd-commit-hash` 後，`master` 分支上最近兩個 Commit 完全消失
- 檔案內容也回到第三個 Commit 時的狀態
- 這個操作是**不可逆**的，除非你之前已經把內容 stash 或 copy 了

### `git revert`：安全的新增反向 Commit

```bash
git revert <commit-hash>
```

`git revert` 的運作方式是：
1. **新增一個全新的 Commit**，這個 Commit 的內容是「指定 Commit 所做的變更之反向」
2. 原始 Commit **仍然保留在歷史中**
3. Branch Pointer 向前移動到新的 Commit

**實作情境**：
- 在 `master` 上做了一個「bad commit」，修改了 `cat.txt` 和 `dog.txt`
- `git revert bad-commit-hash` → Git 新增了一個 Commit，內容是「把 bad commit 的變更全部反向執行」
- 歷史完整保留，只是多了一個「撤銷」的標記

### Reset vs. Revert：核心差異視覺化

```
Reset（改寫歷史）：
  Before:  A → B → C → D (HEAD)
  After:   A → B (HEAD)
           ↘ C → D 被刪除

Revert（安全新增）：
  Before:  A → B → C → D (HEAD)
  After:   A → B → C → D → D' (HEAD, D' 是反轉 D 的新 Commit)
```

### 為什麼在協作場景中必須用 Revert？

這是本節最關鍵的「為什麼」。

**場景**：你和三位同事在不同的機器上共同開發一個專案。

```
你的機器：Commit 1 → Commit 2 → Commit 3（其中 Commit 3 有問題）
同事 A：  Commit 1 → Commit 2 → Commit 3
同事 B：  Commit 1 → Commit 2 → Commit 3 → 她自己的 Commit 4
```

**如果你用 `reset`**：
- 你把 Commit 3 從你的歷史中刪除了
- 但同事 A 和同事 B **已經拿到了 Commit 3**，並且已經基於它做了後續開發
- 當你 Push，並且同事 Pull 時，Git 會遇到嚴重的歷史衝突——**你們的歷史已經完全不同了**
- 這種情況非常難以調和，會造成團隊的困擾

**如果你用 `revert`**：
- 你新增了一個「撤銷 Commit 3」的 Commit
- 這個新 Commit 可以正常 Push
- 同事 Pull 時，Git 只需要「把這個新 Commit 套用到他們的歷史上」，過程簡單且安全
- 歷史完整保留，團隊成員之間的協作不會受到干擾

> **實務建議**：
> - **還沒 Push 的錯誤 Commit**：`git reset` 很安全，因為只有你自己知道
> - **已經 Push 並被其他人取得的 Commit**：**一定要用 `git revert`**

### Revert 的衝突處理

`git revert` 和 Merge 一樣，都可能遇到衝突。當 Git 無法自動判斷「要保留什麼」時：

1. Git 報告衝突
2. 你需要手動開啟衝突檔案，決定最終要保留什麼
3. `git add <file>` 標記為已解決
4. Git 會自動產生一個 Commit（預設訊息為 "Revert <original-commit-message>"）

## 💡 重點摘要

- `git reset <hash>`：**刪除**指定範圍內的 Commit，Branch Pointer 向後移動，不改變 Working Directory
- `git reset --hard <hash>`：**刪除** Commit **並清除** Working Directory 的內容，危險且不可逆
- `git revert <hash>`：**新增一個反向 Commit** 來撤銷指定 Commit 的變更，**歷史完整保留**
- 還沒 Push 的錯誤 Commit → 用 `reset`；已經 Push 的錯誤 Commit → 用 `revert`
- 協作時用 `reset` 會造成團隊歷史衝突，**千萬不要對已分享的歷史使用 reset**
- `revert` 衝突時的處理流程，與 Merge Conflict 完全相同

## 關鍵字

git reset, git reset --hard, git revert, history rewriting, branch pointer, merge conflict, collaboration, git push, git pull, HEAD~n
