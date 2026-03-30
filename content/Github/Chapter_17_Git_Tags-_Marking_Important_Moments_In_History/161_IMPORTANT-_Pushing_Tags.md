# 推送 Tags 到 Remote Repository

## 📝 課程概述

本單元說明一個非常重要但常被忽略的事實：**Tags 在預設情況下不會隨 `git push` 一併推送到 remote**。如果你只推送 commit 而忘記推送對應的 Tag，其他人就無法看到版本發布的標記。本單元介紹兩種正確推送 Tags 的方法，確保團隊成員都能取得完整的版本資訊。

## 核心觀念與實作解析

### 預設行為：Tags 不會自動推送

這裡之所以特別強調這一點，是因為它是**非常多初學者踩過的坑**。你執行了 `git push`，程式碼上去了，但 GitHub 上 Tags 數量顯示為 0：

```
0 tags
```

> 這是 Git 的刻意設計：Tags 通常代表正式的版本發布，是很重要的里程碑。Git 希望你**主動、明確地**選擇是否要分享這些 Tags，而不是預設全部推送。

### 方法一：一次推送一個 Tag

如果你只需要推送特定的一個 Tag，可以指定 Tag 名稱：

```bash
git push origin v17.0.1
```

```
git push <remote名稱> <tag名稱>
```

這適合在正式發布某個版本後，單獨將它推送出去。

### 方法二：一次推送所有 Tags

使用 `--tags` 參數，可以將本地**所有尚未推送到 remote 的 Tags**一次全部推送：

```bash
git push origin --tags
```

或者推送到另一個 remote：

```bash
git push colt --tags
```

> `--tags` 會推送所有本地 Tags，而不是只推當前 branch 上的 Tags。這是個值得注意的細節——如果你的 repo 有多個 branch 上的 Tags，全部都會被推送。

### 兩種推送方式的對比

| 方式 | 命令 | 適用場景 |
|------|------|----------|
| 推送單一 Tag | `git push origin v17.0.1` | 正式發布一個版本 |
| 推送所有 Tags | `git push origin --tags` | 首次設定 repo，或同步所有版本標記 |

### 為什麼 `--tags` 比逐一推送快得多？

推送一個 Tag 的時間幾乎可以忽略不計。但如果你有 141 個 Tags（像 React 專案那樣），逐一推送需要 `141 × RTT`（網路往返時間）。使用 `--tags` 一次推送，Git 會批次處理，節省大量時間。

### Tags 與 Semantic Versioning 的實際應用

在業界，**每一次 Semantic Versioning 的版本發布（Major、Minor、Patch）都應該用 Tag 標記並推送到 remote**。這讓：

- 團隊成員可以快速找到「v2.3.0 是哪個 commit」
- CI/CD 系統可以基於 Tag 觸發建置與部署流程
- 外部貢獻者可以在 GitHub Releases 頁面看到完整的版本歷史

## 💡 重點摘要

- **`git push` 預設不會推送 Tags**，這是 Git 的預設行為，不是 bug
- 使用 `git push origin <tag名>` 推送單一 Tag
- 使用 `git push origin --tags` 一次推送所有 Tags
- **已推送的 Tag 移動後，需要再次執行 `git push -f` 才能更新 remote**，這在團隊協作中應非常謹慎

## 關鍵字

git push, --tags, remote, pushing tags, git push origin, 標籤推送, remote repository, 版本發布, CI/CD
