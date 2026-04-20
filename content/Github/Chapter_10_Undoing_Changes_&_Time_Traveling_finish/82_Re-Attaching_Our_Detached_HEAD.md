# Re-Attaching Our Detached HEAD — Detached HEAD 的實戰操作

## 📝 課程概述

本節深入探討 Detached HEAD 狀態下的三種處理策略：**觀察歷史**（時間倒流查看檔案）、**安全返回**（回到原本的 Branch），以及**以此為起點建立新 Branch**（長期保存時間旅行的成果）。理解這三種操作的時機與方式，是掌握 Git 時間旅行的關鍵。

## 核心觀念與實作解析

### Detached HEAD 的視覺化理解

在 Git 的內部結構中，正常情況下：

```
HEAD ──→ master ──→ Commit D (最新)
              ──→ Commit C
              ──→ Commit B
              ──→ Commit A (最初)
```

當你執行 `git checkout <old-commit>` 時，結構變成：

```
HEAD ──→ Commit B (脫離了 master，直接指向)
master ──→ Commit D (不動)
```

HEAD 不再「附著」在 Branch 上，而是直接懸浮在某個歷史 Commit。這就是 Detached（脫離）的由來。

### 策略一：只是觀察，做完就走

如果你只是好奇「兩週前的程式碼長什麼樣子」，最簡單的做法：

```bash
git checkout abc1234    # 進入 Detached HEAD，查看歷史
# （瀏覽、研究、做筆記⋯⋯）
git switch master       # 回到原本的 Branch，所有歷史完整無損
```

**這個過程不會破壞任何東西**。你只是用「倒帶播放」的模式看了過去的 Commit，播放完畢就回來。

### 策略二：在歷史起點建立新 Branch

如果你不只「觀看」，還想在這個歷史狀態下**長期開發新功能**：

```bash
git checkout abc1234              # 進入 Detached HEAD
git switch -c chapter-two-redo    # 建立並切換到新 Branch
# HEAD 現在重新附著到新 Branch：HEAD → chapter-two-redo → Commit B
```

新 Branch 會以當時的 Commit 為基礎，之後的 Commit 都會長在這條新 Branch 上。原本的 `master` 分支完全不受影響。

### `git switch -`（dash）：快速回到上一個 Branch

當你在多個 Branch 之間來回跳躍後，想回到「上一個」所在的位置：

```bash
git checkout HEAD~1   # 退回到父 Commit
git switch -          # 回到剛才所在的 Branch（無需記住名字）
```

`-` 是一個 Git 的快捷語法，代表「上一次所在的 Branch」。
![[Pasted image 20260416204529.png|700]]
### `HEAD~n` 語法詳解

`~` 是「相對於 HEAD 往前幾代」的語法：

```bash
HEAD~1   # HEAD 的父 Commit（上一個）
HEAD~2   # HEAD 的祖父 Commit（上兩個）
HEAD~n   # HEAD 往前 n 代
```

實驗過程（講師在《了不起的蓋茨比》Repo 中）：
1. 在 `master` 最新 Commit 上，`HEAD~1` = 上一次 Commit（增加了標題的版本）
2. `HEAD~2` = 再往前一個（沒有標題的版本）

> **使用時機**：當你只想退個一兩步，但懶得去查 Hash 時，`HEAD~n` 比 Copy Hash 更快速直觀。

### Detached HEAD 狀態下的 Commit 去了哪裡？

這是最容易令人困惑的部分。假設你在 Detached HEAD 狀態下建立了一個新 Commit：

```
HEAD (detached) ──→ 新 Commit X
master ──→ Commit D（不動）
```

這個 Commit X **存在**嗎？答案是：**存在於磁碟上，但沒有 Branch 指向它**。

- 如果你立即建立一個 Branch（`git switch -c new-branch`），新 Branch 會指向 Commit X，一切正常
- 如果你直接切換走（`git switch master`），**Commit X 會成為「孤兒」**，最終被 Git 的垃圾回收清除

> **風險提示**：千萬別在 Detached HEAD 狀態下做了大量 Commit，然後直接切換分支而不建立 Branch——那些 Commit 會「飄在外太空」，無法再被找回。

## 💡 重點摘要

- Detached HEAD = HEAD 直接指向某個 Commit，而非指向 Branch
- Detached HEAD 狀態本身不是錯誤，是「觀看歷史」的正常模式
- 在 Detached HEAD 中**可以自由建立 Commit**，但切換 Branch 前必須建立新 Branch 來「固定」它
- `git switch -` 是回到上一個 Branch 的快捷鍵，不必記住分支名稱
- `HEAD~n` 語法可以快速定位到 HEAD 往前 n 代的 Commit
- **從 Detached HEAD 直接切走而不建 Branch = 你的 Commit 會永久消失**

## 關鍵字

detached HEAD, git checkout, git switch, HEAD~n, git switch -, orphan commits, branch pointer, time travel, git log
