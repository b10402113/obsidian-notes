# Main & Master — GitHub 預設分支名稱的演變

## 📝 課程概述

本節說明 GitHub 在 2020 年底做出的重大決定：將新倉庫的預設分支名稱從 `master` 改為 `main`。這個改變反映了對種族歧視語言的重視，現在越來越多的專案和組織採用 `main` 作為標準分支名稱。本節的目標是讓你理解這個變化，並學會如何在本地的 `master` 與遠端的 `main` 之間自如切换。

## 核心觀念與實作解析

### 為什麼要從 master 改為 main？

2020 年 10 月，GitHub 宣布新建立的 Repo 預設分支名稱從 `master` 改為 `main`。這是出於對多元包容的考量——`master` 這個術語在歷史上與奴隸制度相關聯，而 `main` 是一個中性、描述性的詞彙，能更好地代表「主要的」或「首要的」含義。

### 新舊 Repo 的分支名稱差異

| Repo 建立時間 | GitHub 預設分支 | 本地 `git init` 預設分支 |
|---------------|----------------|------------------------|
| 2020 年 10 月之前 | `master` | `master` |
| 2020 年 10 月之後 | `main` | `master` |

也就是說：
- **GitHub 新建 Repo**：預設分支是 `main`
- **本地用 `git init` 建立 Repo**：預設分支仍是 `master`

這造成了初期的一些混淆，但兩者都是普通的分支名稱，沒有任何技術差異。

### 如何將本地 master 分支改名為 main

```bash
# 切換到 master 分支
git switch master

# 將分支重新命名為 main
git branch -M main
```

`-M` 是 `--move --force` 的組合，強制重新命名（如果 `main` 已存在，需要先處理）。

### 重新命名後需要做什麼？

1. 推送新的 `main` 分支到 GitHub：
   ```bash
   git push -u origin main
   ```

2. 如果 GitHub 上仍有舊的 `master` 分支，且不再需要：
   ```bash
   git push origin --delete master   # 刪除 GitHub 上的 master 分支
   ```

3. 設定 `main` 為 GitHub 上的預設分支（可選）：
   - 在 GitHub Repo → Settings → Branches → Default branch
   - 將預設分支切換為 `main`

### 為什麼本地仍然預設是 master？

Git（軟體本身）預設使用 `master` 作為初始分支名稱，這與 GitHub 無關，是由 Git 這個工具本身設定的。只要你用的是 `git init`，預設分支名稱就會是 `master`。

如果要改變 Git 的全域預設分支名稱：
```bash
git config --global init.defaultBranch main
```

### `master` 與 `main` 只是名字

最重要的理解：**兩者之間沒有任何技術差異**。`main` 不是特殊的分支，`master` 也不是。它們就是普通的分支，只是名字不同。

- `master` → 一樣是常規分支
- `main` → 一樣是常規分支

改用 `main` 主要是出於**社會與文化層面**的考量，而非技術需求。

### 在同一個 Repo 中同時存在 master 與 main 的情況

如果在 GitHub 上：
- 先推送了 `master` 分支
- 後來又改名為 `main` 並推送

GitHub 上會**同時存在兩個分支**。你可以：
- 继续使用 `master`
- 或進入 Settings 將 `main` 設為預設分支
- 或刪除 `master` 分支

> **業界趨勢**：越來越多的開源專案和公司正在遷移到 `main` 分支名稱。建議新專案直接使用 `main`，舊專案可視情況遷移。

## 💡 重點摘要

- GitHub 在 2020 年將新 Repo 的預設分支從 `master` 改為 `main`
- `git init` 本地預設仍是 `master`，GitHub 新 Repo 預設是 `main`，兩者沒有技術差異
- 將本地分支從 `master` 改名為 `main`：`git branch -M main`
- 重新命名後記得使用 `git push -u origin main` 推送並設定 Upstream
- `master` 和 `main` 只是分支名稱，沒有任何功能上的不同
- 建議新專案直接使用 `main` 作為預設分支名稱，與業界趨勢接軌

## 關鍵字

master, main, default branch, git branch -M, git push -u origin main, git init, GitHub default branch, branch rename, init.defaultBranch
