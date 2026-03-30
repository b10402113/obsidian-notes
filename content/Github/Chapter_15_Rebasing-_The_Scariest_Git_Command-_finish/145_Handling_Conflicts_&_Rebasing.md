# Handling Conflicts & Rebasing — 處理 rebase 衝突

## 📝 課程概述

rebase 和 merge 本質上都是在「把兩條歷史合併在一起」，因此 rebase 同樣可能遇到 conflict。本單元會實際演示當 rebase 途中遭遇 conflict 時，Git 如何中斷 rebase 流程，讓你修復衝突後再用 `git rebase --continue` 繼續完成。**處理的邏輯與 merge conflict 几乎相同，只是 commit 方式不同。**

## 核心觀念與實作解析

### rebase 也會 conflict——原因與 merge 完全一樣

rebase 的過程中，Git 會把你的 feature branch 上的每一個 commit，逐一「播放」到 master 的最新 commit 上。如果其中某一個 commit 試圖修改的程式碼區塊，恰好在 master 上也已經被改過了，就會產生 conflict——這和 merge conflict 的成因是完全一樣的。

### 衝突發生的當下：rebase 會停在衝突的位置

 instructor 在 demo 中做了以下情境：

1. 在 `feat` branch 的 `website.txt` 中修改了「navbar added」→「top navbar added」、「login form」→「log out form」
2. commit 並切回 `master`
3. 在 `master` 的同一個 `website.txt` 中修改了「navbar added」→「main navbar added」、「login form」→「sign up form added」
4. commit 並切回 `feat`
5. 執行 `git rebase master`

Git 嘗試 replay commit：
- 第一個 commit：成功
- 第二個 commit：成功
- 第三個 commit：**conflict 發生！**

Git 的輸出會顯示：
```
CONFLICT (content): Merge conflict in website.txt
Failed to merge in changes.
```

### rebase 停在半路上的狀態

> instructor 特別提醒：此時 rebase 只完成了一部分，**repo 處於一個「進行中」的狀態**。千萬不要慌張，你的 commit 歷史並沒有消失，只是 Git 在等你去處理問題。

執行 `git status` 時，你會看到 Git 明確告訴你：
- 目前在哪一個 commit 上卡住
- 哪一個檔案有 conflict
- 你有兩個選項：**解決衝突後 continue**，或**直接 abort 放棄整個 rebase**

### 如何解決 conflict 並繼續 rebase

這裡的步驟和 merge conflict 幾乎一模一樣：

1. **打開衝突的檔案**，手動決定要保留哪一邊的內容（`<<<<<<< HEAD` 與 `>>>>>>>` 之間的範圍）
2. **編輯並儲存檔案**
3. **`git add <file>`**——將解決後的檔案加入 staging area
4. **`git rebase --continue`**——讓 Git 繼續完成剩餘的 rebase

**關鍵差異**：在 merge 時，你處理完 conflict 後執行的是 `git commit`；但在 rebase 時，因為 Git 不會為你創建 merge commit，所以**不要** `git commit`，而是執行 `git rebase --continue`。

### 整個 rebase 完成的意義

當 rebase 全部完成後，你的 feature branch 上的所有 commits 已經被重新生成，並且完整地「長在了」master 最新 commit 之後。**你和 master 的程式碼內容是同步的，同時歷史結構維持線性，沒有 merge commit。**

### rebase conflict 的實務心態

> **不要因為害怕 conflict 就回避 rebase。** conflict 遲早會遇到，但處理的步驟並不困難，且 Git 會一路清楚指引你：衝突在哪裡、該怎麼繼續或放棄。

與 merge conflict 一樣，rebase conflict 的發生通常預告了一件事：**你和團隊成員在不同的 branch 上修改了同一行程式碼**。這本身不是錯誤，而是協作中的正常現象，只是需要人工來決定最終保留什麼版本。

## 💡 重點摘要

- rebase 途中同樣會發生 conflict，原因是 Git 嘗試 replay commit 時發現同一行程式碼被兩邊修改了
- conflict 發生時，rebase 會暫停，Git 會清楚告知你在哪個 commit、哪個檔案卡住
- 解決 conflict 的步驟：**編輯檔案 → `git add` → `git rebase --continue`**
- **不要執行 `git commit`**，在 rebase 的 conflict 修復後要執行 `git rebase --continue`
- 如果不想繼續，可以執行 `git rebase --abort` 放棄整個 rebase，恢復到 rebase 前的狀態
- conflict 不是 rebase 的缺陷，而是「你和團隊修改了同一處程式碼」這個事實遲早會帶來的正常結果

## 關鍵字

`git rebase`, `git rebase --continue`, `git rebase --abort`, Merge Conflict, Conflict Resolution, Staging Area, Replay, Rebase in Progress
