# 使用 Interactive Rebase 修改 Commit Message（reword）

## 📝 課程概述

本單元實作如何使用 Interactive Rebase 的 `reword` 指令修改歷史中任何一個 commit 的訊息。我們將看到當 commit message 被改變時，該 commit 的 hash 以及所有後續 commit 的 hash 都會隨之更新——這是理解 Rebase 行為的核心觀念。

## 核心觀念與實作解析

### 常見情境：本地分支的 Commit Message 混亂

想像這樣的場景：你在 local 開發一個功能，一開始沒有注意 commit message 規範，隨意寫了：

```
I added project files     ← 過去式 + 主詞，不符合 convention
add project files         ← 正確的 imperative mood
```

等你準備好要開 PR 時，才發現這些訊息要給老闆或開源專案的 maintainer 看。這個時候，你不想再多出一個 commit 說「fix commit message」，而是**直接在歷史上修改它**——`reword` 就是為此而生。

### 實作流程

**Step 1：建立一個實驗分支**

```bash
git switch -c my-feat
```

**Step 2：執行 Interactive Rebase**

```bash
git rebase -i HEAD~9
```

這裡 `9` 代表「我要重建最近 9 個 commit」。起點是 `HEAD~9`，終點是目前的 HEAD。注意這裡**沒有指定要 rebase 到哪個 branch**，純粹是「在原地重建」。

**Step 3：編輯器中修改指令**

編輯器會列出 9 個 commit（ oldest → newest）：

```
pick abc1234 initial commit
pick def5678 I added project files    ← 想改這個
pick ghi9012 add bootstrap
...
```

把 `pick` 改成 `reword`（或簡寫 `r`）：

```
pick abc1234 initial commit
reword def5678 I added project files
pick ghi9012 add bootstrap
...
```

存檔後，Git 會針對這個 commit 再次開啟編輯器，讓你修改 commit message。

**Step 4：修改 commit message**

編輯器直接開啟該 commit 的訊息內容：

```
I added project files
```

改成符合規範的寫法：

```
add project files
```

存檔離開，Rebase 完成。

### Reword 的底層發生了什麼？

> 當你改變了 commit message，Git 並不是「原地修改」了這個 commit，而是**建立了一個新的 commit**，有新的 hash，訊息是你指定的新內容，但資料內容與原 commit 完全相同。

也就是說：
- 舊 commit `def5678` 消失
- 新 commit `xyz7890` 出現，內容與 `def5678` 完全相同，只是訊息不同
- 所有基於 `def5678` 的後續 commit，因為 parent hash 變了，全部變成了新 commit

這就是為什麼我們說 **Rebase 改變的是歷史，而不是檔案內容**。

### 為什麼 `HEAD~N` 的數字要正確？

在第一次 `reword` 完成後，原本的 9 個 commit 變成了 9 個新 commit。如果你再次執行 `git rebase -i HEAD~9`，會得到錯誤——因為從 `HEAD` 往前數 9 個 commit，範圍已經改變了。你需要重新計算正確的數字。

### Reword 與 `git commit --amend` 的差異

| 指令 | 適用範圍 | 行為 |
|------|---------|------|
| `git commit --amend` | 只能改**最近一個** commit | 原地修改 |
| `git rebase -i ... reword` | 可以改**任意過往**的 commit | 建立新 commit，影響後續所有 commit |

兩者都改 commit message，但 `rebase -i` 的影響範圍遠大於 `amend`。

## 💡 重點摘要

- 執行 `reword` 時，Git 會為該 commit 建立一個全新的 commit（新的 hash），但內容不變，只是訊息不同
- 由於每個 commit 的 hash 與其 parent hash 有關，改變任何一個 commit 都會讓所有後續 commit 重新計算 hash
- 使用 `git rebase -i HEAD~N` 時，N 必須是精確的範圍數字，完成後若要再次 rebase，需要重新計算
- 當只需要修改最近一個 commit 時，`git commit --amend` 是更簡單的替代方案

## 關鍵字

`git rebase -i`, reword, `git commit --amend`, commit hash, parent hash, interactive editor, imperative mood, rewrite history
