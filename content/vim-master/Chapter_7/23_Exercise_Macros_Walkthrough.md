# Exercise：Macros 實戰演練

## 📝 課程概述

本單元透過四個實務情境，讓你亲手練習 Vim Macros 的錄製與應用。我們將處理：Python `print` 語句到函式的轉換、使用者帳戶批次操作、日誌資料正規化，以及多行資料合併與 HTML 連結處理。這些都是 DevOps 和日常文字編輯中真實會遇到的問題。

---

## 核心觀念與實作解析

### 實戰一：Python `print` 語句加括號

Python 2 的 `print "text"` 需要轉為 Python 3 的 `print("text")`：

```text
print "Hello world"       →  print("Hello world")
print "Another line"       →  print("Another line")
```

```text
# Macro 策略：在空白處插入 "("，在行尾追加 ")"
qa
0                 # 標準化：移到行首
f<Space>          # f + 空白 → 跳到第一個空白處
r(                # r = replace one char → 將空白替換為 "("
A                 # Append → 移到行尾並進入插入模式
)<Esc>            # 追加 ")" 並返回 Normal mode
j                 # 移到下一行（方便重複）
q

# 執行
@a                # 第一次
@@                # 重複上一次 Macro
@@                # 再執行一次
```

> 💡 這裡有多種等效做法，例如用 `t"`（跳到引號前）代替 `f<Space>`。**方法不止一種，結果正確即可。**

### 實戰二：批次系統管理操作

情境：對 Linux 使用者帳戶執行相同操作，並將結果寫入日誌檔案：

```text
# 每行是一個使用者帳戶名稱
alice
bob
carol
```

```text
# Macro 策略：生成 passwd + 追加寫入檔案的完整指令
qb
yw                 # yank a word → 複製當前行的使用者名稱
Ipasswd -l <Esc>   # 在行首插入 lock 命令
A 2>> locked_users.txt<Esc>  # 在行尾追加寫入指令
j
q

# 執行（3 個使用者）
4@b                # 執行 4 次（或根據實際行數調整）
```

> 💡 `passwd -l` 是 lock user account，`2>>` 將 stderr 追加到日誌檔，`&` 將命令放到背景執行。

### 實戰三：日誌資料正規化為 CSV

情境：將 Linux 防火牆日誌中的雜訊過濾，只保留時間戳、來源 IP 和目標連接埠：

```text
# 原始日誌（經常是單行長字串）
Mar 15 09:23:41 server kernel: [UFW BLOCK] IN=eth0 OUT= SRC=192.168.1.100 DST=10.0.0.1 PROTO=TCP SPT=443 DPT=8080 ...

# 目標輸出
192.168.1.100, 8080
```

```text
# Macro 策略：逐步刪除雜訊，替換關鍵分隔符
ql
0                           # 移到行首
tw                           # t w → 跳到空白後（時間戳後）
d/C=</Enter                  # d/ + 搜尋 "C=" → 刪除到 C= 前
dw                           # 刪除 "C=" 中的 "C"
r,                           # r = replace → 把 "=" 換成 ","
f<Space>                     # 跳到 IP 後的空白
d/DPT=</Enter                # 刪除到 DPT= 前
dw                           # 刪除 "DPT"
r,                           # 把 "=" 換成 ","
f<Space>                     # 跳到連接埠後的空白
D                            # 大寫 D → 刪除到行尾
j
q

# 驗證 Macro 內容
:reg l

# 執行
@l   # 先確認第一筆正確
@@@  # 重複直到所有行完成
```

### 實戰四：多行合併為單行

情境：將國家名稱和人口數分開兩行的資料，合併到一行並用分號分隔：

```text
# 原始格式
Japan
125 million
Germany
83 million

# 目標格式
Japan; 125 million
Germany; 83 million
```

```text
# Macro 策略：刪除國家，移到下一行剪下數字，貼回原行，替換空白為分號，刪除多餘行
qc
0
dw                  # 刪除國家名稱
j
dW                  # 大寫 W = 剪下下一行的數字（不截斷標點）
k
P                   # 大寫 P = 貼到游標前（數字在國家名稱前）
r;                  # r = replace 1 char → 把空白換成 ;
j
dd                  # 刪除空出的下一行
q

# 執行（根據資料筆數）
4 @c
```

### 實戰五：HTML 連結轉換為純網域

情境：將 `<a>` 標籤內的文字，只保留網域部分：

```text
# 原始
Visit <a href="http://spam-a-lot.com">spam-a-lot.com</a> now!
Visit <a href="http://army-spy.com">army-spy.com</a> now!

# 目標
spam-a-lot.com
army-spy.com
```

```text
# Macro 策略：刪除到 @，再改變 closing tag 為 inner
qq
df@                 # d f @ → 刪除到第一個 @（含）
f<                  # 跳回 < 符號
cf>                 # c f > → 改變從游標到 > 的內容（相當於刪除並進入插入模式）
<Enter><Esc>        # 刪除 closing tag 並返回
q

# 執行
@q
@@
@@ ...
```

> 💡 這裡用 `f<` + `cf>` 的組合，本質上是把 closing tag `</a>` 轉換成一個「空內容」的 `>`，再以 inner 方式處理。

### 實戰六：批次應用 Macro 到大量行——`:[range] normal @[register]`

當資料筆數很多，手動 `@@` 不切實際時：

```text
# 開啟行號顯示
:set nu

# 假設電話號碼從第 25 行到第 73 行
:25,73 normal @p

# 或對整個檔案
:1,$ normal @p
:% normal @p         # 簡寫
```

> 💡 `:%` 這個 range 代表「檔案全部行」，是 `1,$` 的常用簡寫。

---

## 💡 重點摘要

- **Macro 的本質是「按鍵重放」，所以理論上任何能在 Vim 中執行的操作都可以錄進 Macro。**
- **日誌正規化（實戰三）是 Macro 最典型的實務應用場景之一：刪掉不要的、保留要的就是核心邏輯。**
- **`.vimrc` 中 `let @[r] = '...'` 的語法讓你能直接用字串定義 Macro，避免手動輸入特殊字元的麻煩。**
- **`:[range] normal @[register]` 讓你優雅地對大量行應用同一個 Macro，比 count 參數更精確。**
- **隨時用 `:reg [register]` 確認 Macro 內容，是 Debug 時最可靠的方法。**

---

## 🔑 關鍵字

Log Parsing, CSV, normal, Macro, Register, Append, Repeat
