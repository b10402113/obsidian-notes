# Semantic Versioning 語義化版本規範

## 📝 課程概述

本單元介紹 Semantic Versioning（語義化版本），這是一套用於賦予軟體版本號意義的公開規範。版本號的 `X.Y.Z` 三個數字並非任意決定，而是各自承載了「破壞性變更」、「新功能」與「修補」這三種不同層級的變更資訊。理解這套規範，才能正確使用 Git Tags 標記有意義的發布點。

## 核心觀念與實作解析

### 為什麼需要語義化版本？

在沒有統一版本規範的時代，`v2.3.4` 對不同人可能代表完全不同的意思。Semantic Versioning 的出現就是為了解決這個問題——讓版本號本身就是一種**有意義的訊息**，工程師只要看到版本數字，就能立即判斷這次更新是否安全、是否需要改動既有程式碼。

### 版本號的結構：`MAJOR.MINOR.PATCH`

```
major.minor.patch
  ↑      ↑      ↑
主要     次要     修補
版本     版本     版本
```

#### Patch（最右側數字）— 修補版本

當你進行**錯誤修正（Bug Fix）**或小幅調整，但**不新增功能也不做任何破壞性變更**時，就增加這個數字。

- 使用者不需修改任何程式碼即可直接升級
- 範例：`1.0.0` → `1.0.1`：修正了一個小 bug

#### Minor（中間數字）— 次版本

當你**新增功能（New Features）**且**完全向後相容（Backwards Compatible）**時，就增加這個數字，並將 Patch 歸零。

- 新功能是**可選的**，使用者不必非得用
- 範例：`1.0.1` → `1.1.0`：新增了一個選用的新功能

#### Major（左側數字）— 主版本

當你做了**破壞性變更（Breaking Changes）**時，就增加這個數字，並將 Minor 與 Patch 同時歸零。

- 可能移除功能、大幅改變 API、使用方式與舊版不相容
- 通常需要寫專文說明遷移方式
- 範例：`16.13.x` → `17.0.0`：React 17 的重大更新

### 版本號演進範例

```
1.0.0  →  初始公開發布
1.0.1  →  Patch：修正 Bug
1.0.2  →  Patch：再次修正
1.1.0  →  Minor：新增可選功能
1.2.0  →  Minor：再新增功能
2.0.0  →  Major：破壞性變更，須大幅修改
```

### Semantic Versioning 與 Tags 的關係

在 Git Tags 的應用場景中，我們使用 Semantic Versioning 來命名 Tag：

- `v1.0.0`、`v2.3.4`、`v17.0.1` 都是典型的 Semantic Versioning 命名
- 注意：`v` 字首**並非 Semantic Versioning 規範的一部分**，但業界普遍習慣使用（規範只定義 `1.0.0` 這樣的格式）

### 前置版本標記（Pre-release）

規範也允許在版本號後附加前置標記：

```
17.0.0-beta.1    # beta 版本
16.0.0-alpha.5   # alpha 版本
```

這些標記表示該版本尚未正式發布，供測試使用。

## 💡 重點摘要

- Semantic Versioning 讓版本號 `major.minor.patch` 的三個數字各自承載明確意義：**破壞性變更 / 新功能 / 修補**
- 每次 Minor 發布後，Patch 歸零；每次 Major 發布後，Minor 與 Patch 同時歸零
- 這個規範與 Git Tags 緊密結合，**每一次有意義的版本發布都應該用 Tag 標記**

## 關鍵字

Semantic Versioning, Major Version, Minor Version, Patch Version, Backwards Compatible, Breaking Changes, Pre-release, v-prefix, 版本規範
