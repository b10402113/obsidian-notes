# Comparing Merging & Rebasing — merge 的歷史困境與 rebase 的解法

## 📝 課程概述

本單元從一個真實場景出發：當你獨自在一條 feature branch 上開工時，團隊的其他成員正持續往 `master` 分支推送新 commit。你不斷需要將 `master` 的變更同步到自己的 feature branch——而每次這樣做，都會產生一個 **merge commit**。當專案夠大、開發人數夠多時，你的 commit 歷史會被這些無意義的 merge commit 淹沒。`git rebase` 就是為了解決這個問題而存在的替代方案。

## 核心觀念與實作解析

### merge 的問題：沒有意義的 merge commit 讓歷史變髒

我們先用一個具體情境來理解問題：

> 你在 feature branch 上開發，工作了兩天。過程中，五位同事陸續完成任務並 merge 進 `master`。每一次你要同步 `master` 的新變更，都只能 `git merge master`，然後 Git 產生一個 merge commit。

這個 merge commit 的訊息通常是 `Merge branch 'master' into feat` 之類的東西——**它沒有描述任何實質的程式碼變更，它只是告訴你「我把 master 合併進來了」。**

結果是：
- 你的 feature branch 充滿了這些「不是我寫的 commit」
- 別人要 review 你的程式碼時，得穿過一堆 merge commit 才能找到你真正的 commit
- 如果團隊有 10 個人，每個人都這樣做，整個 `master` 的歷史就會變成一鍋粥

### 用 merge 會發生什麼事

當你在 `feature` branch 上執行 `git merge master`，Git 會：

1. 嘗試 fast-forward（如果可以的話）
2. 如果不行（你的 branch 已經落後了），就建立一個 **merge commit**，這個 commit 有**兩個 parent**：一個指向你原本的 feature 最新 commit，一個指向 `master` 的最新 commit

這個 merge commit 本身沒有程式碼變更，純粹是「膠水」，用來把兩條歷史黏在一起。**當這樣的膠水 commit 越來越多，歷史就越來越難讀。**

### rebase 的解決方案：改變歷史的「起點」

rebase 的精神就藏在它的名字裡：**re-base = 重新給定基底**。

> 當你對 `feature` branch 執行 `git rebase master`，你的 feature branch 的「起點」就會被搬到 `master` 的最頂端（最新 commit）。

**這裡是 rebase 最重要、也最讓人困惑的一點：它會產生全新的 commit。** Git 會把你原本在 feature branch 上的每一個 commit，重新在 `master` 的最新 commit 之上「播放」一次，生成新的 commit。這些新 commit 的訊息可能相同，但 commit hash（以及儲存的底層指標）全部都不一樣了。

### merge vs. rebase 的圖解對比

```
使用 merge 的結果（有 merge commit）：

  master:  A — B — C ———— E ———— G
                  \       /    \
  feature:        D1 — D2 —      F1 — F2 — F3
                             ↑ merge commit
                             ↑ merge commit

使用 rebase 的結果（線性結構）：

  master:  A — B — C ———— E ———— G

  feature:                  D1' — D2' — F1' — F2' — F3'
                              ↑ 重新播放的全新 commit
```

注意 rebase 後的 `feature` 雖然看起來變成了「在 master 後面」，但：
- `feature` branch 上的 commit 內容（程式碼變更）**完全一樣**
- 只是它們現在「長在」`master` 的最新 commit 之後
- 歷史從交錯變成了**線性**，不再有 merge commit

### 為什麼 rebase 的歷史更好讀？

> 如果我們用 rebase，別人一眼就能看出：「這三個 commit 是這個人在 feature branch 上做的。」他們不需要穿過一堆 merge commit 才能找到實質工作。

當你用 rebase：
- 所有你做的 commit 都**連續排列**在一起
- `master` 的 commit 和你的 commit 交替出現（`A master → B master → C feat → D feat`），而不是交錯（`A master → B master → merge → C feat → merge → D feat`）
- 在有大量 concurrent 開發的專案裡，這種區分讓 code review 和歷史追蹤都輕鬆很多

### rebase 的風險：歷史被「重寫」了

> **Rebase 的代價是：我們正在改寫歷史。** 那些原本的 commit 還在，但變成了新的 commit。這聽起來有點嚇人——實際上也確實需要小心。

正因為 rebase 會產生全新的 commit，這就帶出了我們下一個影片要詳細探討的「**Golden Rule of Rebasing**」。

## 💡 重點摘要

- `git merge master` 在分支已經分歧時會產生 merge commit，這些 commit 沒有程式碼變更，只是「膠水」
- 當團隊活跃、merge 頻繁時，你的 feature branch 會充滿這些沒有資訊價值的 merge commit
- `git rebase master` 把你的 feature branch 的起點搬到 `master` 的最頂端，產生**線性歷史**
- rebase 會**重新產生新的 commit**（新的 hash），而不是更新原有的 commit
- **千萬不要 rebase 別人已經有的 commit**——這是 rebase 最核心的限制

## 關鍵字

`git merge`, `git rebase`, Fast-forward Merge, Merge Commit, Feature Branch, Linear History, Rewriting History, Commit Hash, Branch Divergence
