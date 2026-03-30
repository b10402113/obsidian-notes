# Rebasing Demo — 實際操作 git rebase

## 📝 課程概述

本單元是整個章節的核心實作。我們先用 `git merge` 實際走一遍「同步 master → 產生 merge commit」的流程，感受歷史被汙染的過程；接著再在同一個 repo 上執行 `git rebase`，親眼見證線性歷史是如何形成的。重點在於理解：**rebase 的當下，Git 其實正在為你「重新播放」每一個 commit。**

## 核心觀念與實作解析

### 先用 merge 建構一個充滿 merge commit 的歷史

 instructor 在 demo repo 中做了以下操作：

1. 從 `master` 分支出 `feat`
2. 在 `feat` 上做三個 commit（`begin feature`、`work on feature`、`more work on feature`）
3. 回到 `master`，模擬同事新增 `add footer` 並 commit
4. 切回 `feat`，執行 `git merge master` → **產生第一個 merge commit**
5. 繼續在 `feat` 上做 commit
6. 再回 `master`，模擬同事新增 `add login form` 並 commit
7. 再次在 `feat` 上執行 `git merge master` → **產生第二個 merge commit**

執行完後，`git log` 的輸出會像這樣：

```
commit F3  more work on feature
Merge branch 'master' into feat   ← 多餘的 merge commit
commit E    add login form
Merge branch 'master' into feat   ← 多餘的 merge commit
commit C    add footer
commit B
commit A
```

**這些 merge commit 沒有描述任何你實質在做的事情。** 它們只是讓 Git 知道「我把這兩個分歧的分支接起來了」。當 feature branch 開發數週、有數十次這樣的 merge，你的歷史就面目全非了。

### 終於執行 rebase：`git rebase master`

> 這裡的語法是：先切到你要 rebase 的 branch，然後執行 `git rebase <你想以它為新基底的其他 branch>`。

在 `feat` branch 上執行 `git rebase master` 的過程：

```
# Git 輸出：
Rewinding head to replay your work on top of it...
Applying: begin feature
Applying: work on feature
Applying: more work on feature
```

**「Replay your work on top of it」**——這句話精確地描述了 rebase 在做的事：把你的 feature commits，一個一個「重新播放」在 `master` 的最新 commit 之上。**不是複製，不是搬移，而是重新生成。**

### rebase 的關鍵副作用：commit hash 全部變了

 instructor 在 demo 中特別強調了這點。rebase 前後，`master` 上的 commit hash 完全不變（`2be...`、`34f...`、`e19...`），但 `feat` 上的三個 commit 全部變成了**全新的 commit**：

- 原本 `begin feature` 的 hash 是 `889...`
- rebase 後變成了 `907...`（全新 hash）
- 訊息相同、日期相同、甚至內容也相同，但**它們是嶄新的 commit**

> **Rebase = 重新生成歷史，不是更新歷史。**

這就是 rebase 與 merge 最根本的不同。merge 只是把兩條歷史黏在一起，維持所有 commit 的完整性；rebase 則是把你的 commit 們「Rebuild」在另一個起點上。

### rebase 的語法：千萬別搞混方向

 instructor 觀察到很多學生在這裡會搞混：

```
git rebase master
```

**這句話的意思是：「我要把我的 branch（當前 branch）rebase 到 master 之上。」** 不是把 master 拿來 rebse，更不是把 master rebse 到你的 branch 裡。

你**不會**改變 `master` branch 的任何東西——只有你**當前所在的 branch**（執行 rebase 的那個 branch）的 commit 會被重新生成。

### rebase 完後 branch 結構長怎樣？

> 兩個 branch 依然存在，因為 branch 只是個指標（pointer）。但歷史結構從交錯變成了線性。

rebase 前：
```
  master:  A — B — C ———— E ———— G
                  \       /    \
  feat:           D1 — D2 —      F1 — F2 — F3
                             ↑ merge commits
```

rebase 後：
```
  master:  A — B — C ———— E ———— G

  feat:                  D1' — D2' — F1' — F2' — F3'
                         ↑ 全新 commit，緊接在 master tip 之後
```

現在 `feat` 看起來就像一條從 `master` 最新 commit 直接延伸出去的一條線。**沒有 merge commit，所有 commits 都是「實質 commit」。**

### 你可以完全用 rebase 取代 merge

 instructor 在 demo 的最後示範了這個 workflow：

> 如果你在整個開發過程中都用 rebase 而非 merge，你的 feature branch 會一直保持線性結構。每次 master 有新 commit，你就 `git rebase master`，然後你的 commits 會自動被「Rebuild」到新的 master tip 上。

這個 workflow 的好處是：當你終於準備好要把 feature merge 回 master 時，**你只需要一次 `git merge feat`**（通常可以做 fast-forward），歷史乾淨無比。別人在 `master` 的 log 裡看到的，是一條乾淨的、連續的 commit 線。

## 💡 重點摘要

- 在 feature branch 上執行 `git rebase master`，意思是「以 master 的最新 commit 為新的起點，重新生成我的 feature commits」
- rebase 的輸出 `Rewinding head to replay your work on top of it` 精確描述了它的運作方式
- rebase 後 commit hash 會改變——它們是全新的 commit，不是更新
- `git rebase master` **不會改變 master branch**，改變的是你當前所在的 branch
- rebase 可以完全取代 merge 的使用場景，讓歷史保持線性，最後再用一次 merge（或 fast-forward）合併回主線

## 關鍵字

`git rebase`, `Rewind Head`, Replay, Commit Hash, Feature Branch, Linear History, Fast-forward, Branch Pointer, Git Workflow
