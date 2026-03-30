# 移動（Force）與刪除 Tags

## 📝 課程概述

本單元說明兩個進階的 Tag 操作：使用 `-f` 強制參數將已存在的 Tag 移動到另一個 commit，以及使用 `-d` 參數刪除不再需要的 Tag。這些操作屬於「必要但少用」的工具，正確使用可以修正失誤或清理不再有意義的標記。

## 核心觀念與實作解析

### 移動（Force）一個已存在的 Tag

有時候你可能不小心建立了 Tag，或者想要將某個 Tag 指向不同的 commit。這時你就需要移動 Tag。

**直接覆蓋同名 Tag 會失敗**：

```bash
git tag v17.0.3 532FBabc
```

Git 會輸出錯誤：
```
fatal: tag 'v17.0.3' already exists
```

**解決方法：使用 `-f` 強制參數**：

```bash
git tag -f v17.0.3 532FBabc
```

> 這裡的 `-f` 與 `git branch -f` 的用法一致，都是用來「強制覆寫」的參數。

執行後 Git 會告訴你：
```
Updated tag 'v17.0.3' (was abc123)
```

**注意**：移動已發布的 Tag（特別是團隊共用的 Tag）是一個**危險的操作**，應盡量避免。只有在確認沒有其他人已經基於那個 Tag 進行工作時，才使用 `-f`。

> 我們之所以提供這個功能，是因為難免會有建立失誤的情況。但在正常開發流程中，一旦 Tag 已經推送出去，就**不應該再移動它**。

### 刪除 Tags

刪除 Tag 使用 `-d` 參數（delete），語法與刪除 branch 完全一致：

```bash
# 建立一個 Tag 準備刪除
git tag deleteme

# 刪除它
git tag -d deleteme
```

成功刪除後，Git 會輸出：
```
Deleted tag 'deleteme'
```

**同一個 commit 上可以有多個 Tags**，刪除其中一個不會影響其他的 Tag。這一點與 Branch 不同——兩個 Branch 指向同一個 commit，刪除一個並不影響另一個，Tags 也是同理。

```bash
# 假設目前這個 commit 同時有 v17.0.3 和 deleteme 兩個 Tags
git tag -d deleteme
# 刪除後，v17.0.3 仍然存在
```

## 💡 重點摘要

- 嘗試用相同名稱建立第二個 Tag 會失敗，Git 不允許 Tag 名稱重複
- 使用 `git tag -f <tagname> <commit>` 可以**強制將現有 Tag 移動到另一個 commit**
- 使用 `git tag -d <tagname>` 可以刪除不再需要的 Tag
- **已推送給團隊的 Tag 不應再移動或刪除**，會造成協作者之間的混乱

## 關鍵字

git tag -f, force tag, git tag -d, delete tag, -f flag, -d flag, tag movement, 強制覆寫
