# 活用 Reflog 語法：將 `HEAD@{n}` 應用於各種 Git 命令

## 📝 課程概述

本單元深入 `HEAD@{n}` 這個 reflog 語法，示範如何將 Reflog 中的 entry 當作普通 commit reference 一樣傳給其他 Git 命令使用。你會理解它與 `HEAD~n`（波形符號）在語意上的根本差異——前者指向「過去在 reflog 中的位置」，後者指向「祖先 commit」。學會這個技巧後，下一課就能用它來實際搶救「消失」的 commit。

## 核心觀念與實作解析

### `HEAD@{n}` 語法解析

上一課提到，reflog 中的每一筆 entry 都可以用 `HEAD@{n}` 來表示，這是一種**引用 reflog 特定 entry 的語法**：

```bash
HEAD@{0}   # reflog 中最近一次操作（等同於目前的 HEAD）
HEAD@{1}   # 倒數第二次操作
HEAD@{5}   # 五次操作之前的狀態
```

這個語法**可以像普通 commit hash 一樣傳給其他 Git 命令**——這就是「Passing Reflog References Around」的精髓。

---

### `HEAD@{n}` 與 `HEAD~n` 的根本差異

這是本單元最關鍵的概念對比。很多同學容易把兩者搞混，以為它們「差不多」——但它們指向的東西可能完全不同。

| 語法 | 意義 | 查詢方向 |
|---|---|---|
| `HEAD~n` | **沿 commit graph 向祖先走 n 代** | 永遠沿 parent 關係往回 |
| `HEAD@{n}` | **reflog 中倒數第 n 步的 HEAD 位置** | 沿操作時間軸往回 |

**為什麼這個差異很重要？**

講師親自示範了一個實例：他目前在 `donkey` 分支的 commit `7c1` 上，而 `HEAD@{2}` 也指向同一個 commit `7c1`。但執行 `git checkout HEAD~2`（祖父 commit）則會帶你到完全不同的 `FFE` commit。

這是因為在 reflog 的兩個 entry 中間，夾著一個「進入 detached HEAD 模式後又離開」的操作：整個過程的起點和終點都是同一個 commit，所以 `HEAD@{2}` 和當前 `HEAD` 指向相同位置。

> **`HEAD@{n}` 不一定代表「更舊的 commit」，它代表的是「那個當下 HEAD 在哪裡」。** 這可能是：同一個 commit（只是中間經過了其他分支）、不同的分支、或真正的舊 commit。

---

### 實際應用：把 `HEAD@{n}` 傳給其他命令

這才是這個語法真正發揮價值的地方。你可以把它當成一般 commit 識別符來使用：

**用法一：切換到 reflog 中的歷史位置**

```bash
git checkout HEAD@{2}
```

這會讓你進入 detached HEAD 狀態，指向兩次操作前 HEAD 所在的 commit。講師在影片中執行這個命令後，Git 把他帶到了 commit `D8363`——這就是 reflog 中倒數第二步的實際 commit。

**用法二：用 diff 比對 reflog 狀態**

```bash
git diff HEAD HEAD@{5}
```

這個命令可以比較「現在」和「五次操作前」的差異。講師實際執行後看到兩者之間的變化（因為 `HEAD@{0}` 和 `HEAD@{2}` 恰好是同一個 commit，所以 diff 為空）。

**用法三：配合任意 branch reference**

這個語法不限於 `HEAD`，任何有 reflog 的 reference 都可以：

```bash
master@{5}    # master 分支 tip 五次移動前的狀態
donkey@{1}    # donkey 分支前一次更新時的 commit
```

---

### 一個常見的誤解場景

講師特別點出一個容易讓人困惑的情境：

> 如果你一直停留在同一個 commit 上，只是來回切換了幾個分支（例如：`master → develop → master → feature → master`），那麼即使過了很長時間，`HEAD@{3}` 和 `HEAD@{0}` 很可能都還是同一個 commit。

這就是為什麼 `HEAD@{n}` 不等於「時間旅行到 n 天前的 commit」——它只代表「你在 reflog 中走了 n 步到達的位置」。

下一課會介紹的**時間限定符**（如 `HEAD@{yesterday}`、`HEAD@{1 week ago}`）才是真正與時間掛鉤的語法，能做到更直覺的時間回溯。

## 💡 重點摘要

- **`HEAD@{n}` 語法讓你可以「引用」reflog 中的任意 entry，並像普通 commit 一樣傳給其他 Git 命令使用。**
- **`HEAD~n` 是沿 commit graph 的祖先關係往回走；`HEAD@{n}` 是沿 reflog 的操作記錄往回走——兩者語意完全不同。**
- **`HEAD@{n}` 指向的可能是同一個 commit（當中間經過其他分支切換時），不一定是更舊的 commit。**
- **可以用 `git diff HEAD HEAD@{5}` 等方式，把 reflog 狀態比對當前狀態，這對分析操作歷史非常有用。**
- **時間限定符（如 `HEAD@{yesterday}`）才是真正綁定時間的語法，會在下一課中介紹。**

## 關鍵字

`HEAD@{n}`、reflog reference、passing around、`HEAD~n` vs `HEAD@{n}`、`git checkout HEAD@{n}`、`git diff HEAD@{n}`、branch reflog syntax、time-based qualifier
