# 建立 Lightweight Tags 與 Annotated Tags

## 📝 課程概述

本單元說明如何建立兩種不同類型的 Git Tags：輕量的 Lightweight Tag 以及包含完整 metadata 的 Annotated Tag。理解兩者的差異與適用場景，能幫助你在真實專案中選擇正確的 Tag 類型，特別是當你需要為團隊或開源專案建立可驗證的版本標記時。

## 核心觀念與實作解析

### 建立 Lightweight Tag

Lightweight Tag 是最簡單的形式，只需要一行命令：

```bash
git tag v17.0.2
```

這個命令會讓 Tag `v17.0.2` 指向**當前 HEAD 所指的 commit**（也就是目前 branch 的最新 commit）。

**執行後幾乎沒有任何輸出**，但如果你執行 `git tag`，就會看到新 Tag 出現在列表中。

```
HEAD (master branch)
  ↓（指向）
最新 commit ← v17.0.2 (Tag)
```

> 我們這裡之所以可以直接 `git tag` 而不需指定 commit，是因為預設就是指向 HEAD。這在你想快速標記當前版本的時候非常方便。

### 建立 Annotated Tag

Annotated Tag 多了 `-a` 參數，執行時會開啟文字編輯器，要求你填寫 Tag Message：

```bash
git tag -a v17.1.0
```

與 `git commit` 類似，Git 會提示你輸入訊息。完成後，Tag 會包含：

- Tag 訊息（相當於 commit message）
- 作者名稱與 Email
- 建立日期

**你也可以使用 `-m` 參數直接在同一行填寫訊息**，跳過編輯器：

```bash
git tag -a v17.1.0 -m "Minor release: new feature added"
```

> 這就像 `git commit -m` 一樣，是懶人用法。在 CI/CD 自動化流程中特別實用。

### 為什麼大型專案偏好 Annotated Tags？

因為 Annotated Tags 包含完整的 metadata，且**可以被 GPG 簽署**。這意味著：
- 任何人都能驗證這個 Tag 確實由特定作者建立
- 無法偽造他人的 Tag
- 在開源協作中，這是建立信任的關鍵

### 查看 Tag 詳細資訊：`git show`

使用 `git show <tag-name>` 可以查看 Tag 的完整資訊，包括作者、日期、訊息，以及它所指向的 commit：

```bash
git show v17.0.0
```

輸出範例：
```
tag v17.0.0
Tagger: Colt Steele
Date:   Thu Oct 15 14:30:00 2020

bump versions for 17

commit 532FBabc123... (HEAD -> master, tag: v17.0.0)
Author: Dan Abramov <dan@reactjs.org>
Date:   Wed Oct 14 09:00:00 2020

    bump versions for 17
```

### 為歷史上的舊 commit 建立 Tag

到目前為止，我們都是在 HEAD（即最新 commit）上建立 Tag。但你也可以對過去的任意 commit 建立 Tag，只需指定 commit hash：

```bash
git log --oneline          # 找出目標 commit hash
git tag mytag 532FBabc     # 為這個舊 commit 建立輕量 Tag
```

```
較舊的 commit ← mytag (Tag) ← 新 commit ← HEAD
```

> 這裡的 `mytag` 並不是版本號，只是為了示範 Tag 的命名可以是任意字串。這在你想對某個重要的歷史節點做標記時很有用。

## 💡 重點摘要

- `git tag <name>` 建立 Lightweight Tag，預設指向當前 HEAD
- `git tag -a <name>` 或 `git tag -a <name> -m "message"` 建立 Annotated Tag
- `git show <tag>` 可查看 Tag 的完整 metadata 與指向的 commit
- 大型開源專案**偏好 Annotated Tags**，因為它們可被 GPG 簽署驗證
- Tag 可以附加在任何 commit 上，不限於最新 commit

## 關鍵字

git tag, Lightweight Tag, Annotated Tag, -a flag, -m flag, git show, GPG signing, metadata, commit hash, Tag message
