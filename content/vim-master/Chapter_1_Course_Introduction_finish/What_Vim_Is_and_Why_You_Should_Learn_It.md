# Vim 是什麼？為什麼你應該學會它

## 📝 課程概述

本單元回答兩個最根本的問題：「Vim 是什麼？」與「為什麼值得學？」老師從 Vim 的歷史起源談起，逐步說明 Vim 無所不在的滲透力、超高的編輯效率，以及一旦學會後能在各種環境（終端機、編輯器、甚至 Gmail）自由遷移的知識複利效應。

---

## 核心觀念與實作解析

### Vim 的身世：VI Improved

Vim 的全名是 **VI Improved**，意即「改良版 VI」。VI 是早期 Unix 系統上的文字編輯器，最初起源於更古老的 X 行編輯器（Line Editor）中的 Visual Mode。

> **重要的事實：** 在現代系統上，無論你執行 `vi` 還是 `vim` 命令，真正啟動的幾乎都是 Vim。VI 本身幾乎已被 Vim 完全取代。

---

### 為什麼 Vim 無所不在？

Vim 預設安裝在幾乎所有 Linux 發行版上，而 Nano 或 Emacs 則不一定保證存在。這帶來一個非常實際的好處：

- **登入任何 Linux 伺服器時，你幾乎一定可以使用 Vim。** 若系統唯一可用的編輯器就是 Vim，至少要會基本操作。
- **很多系統工具的外部編輯器預設為 Vim。** 例如 `crontab -e`、`git commit`、`sudo visudo` 等，執行後彈出的就是 Vim 介面。

若只會開啟 Vim卻不知道如何離開（`:q!`），那才是真正的噩夢——老師在這裡特別開玩笑地暗示了這個常見的「Vim 新手困境」。

---

### Vim 的強大之處：遠勝 Nano

如果你是從 Nano 轉過來的學習者，初次接觸 Vim 會有種「從腳踏車升級到機車」的感受。Vim 內建的強大功能包括：

- **Macros** — 錄製並重放一連串操作
- **Registers** — 多重剪貼簿，可同時存放多段文字
- **Text Objects** — 以語義為單位的選取與操作
- **Global Substitution** — 跨檔案一次取代
- **Autocompletion、Searching、Filters** 等

這些功能讓複雜的文字編輯任務也能快速完成。**一旦學會 Vim 的操作邏輯，你會比使用 Nano 時更有效率。**

---

### Vim 知識的遷移力：觸類旁通

Vim 最大的長期價值之一，是它養成的操作習慣幾乎到處都能用：

1. **終端機工具** — `man` 頁面、`less` 等 pager 程式的導航鍵綁定與 Vim 完全一致
2. **Shell 命令列** — 可將 Bash（以及 Zsh、Fish 等主流 Shell）設定為 Vim 模式編輯，例如 `set -o vi`，就能用 Vim 鍵盤快捷鍵瀏覽命令歷史並直接編輯當前命令列
3. **Gmail** — 啟用鍵盤快捷鍵後，許多操作就與 Vim 命令相同（`j`/`k` 上下移動等）
4. **其他編輯器** — Atom、Eclipse、Sublime Text、Notepad++、Xcode 等主流編輯器都支援 Vim Mode 外掛

> **為什麼遷移力很重要？** 學習 Vim 不只是學一個編輯器，而是學一套在任何環境都能適用的「文字操作語言」。

---

### Vim 如同一門語言：Verb + Noun 思維

這是本課程最重要的設計理念。Vim 的命令系統並非一堆雜亂的快捷鍵，而是一套有結構的語法：

| 組成 | Vim 元素 | 範例 | 含義 |
| --- | ------- | ---- | ---- |
| 動詞（Verb） | 操作指令 | `d`（delete）、`c`（change）、`y`（yank） | 做什麼動作 |
| 受詞（Noun / Object） | 文字物件 | `iw`（inner word）、`i"`（inner quote） | 作用在什麼範圍 |

**實際範例：**

- `d3w` = delete three words（三個 word）
- `ci"` = change inside quotes（變更引號內所有內容）
- `dap` = delete a paragraph（刪除一整段）

> **為什麼這個設計厲害？** 你不需要為每種情況死背一個完整命令。你只需要學會「動詞 + 受詞」的組合原則，就能像造句一樣自由組合。**這就是為什麼學 Vim 不是在背命令，而是在學一種語言。**

---

### Syntax Highlighting：不只是美觀

很多人以為 Syntax Highlighting 只是「讓程式碼看起來漂亮」，但對老師來說它最重要的功能是：**快速發現錯誤。**

> 如果你修改了某一行，結果該行的 Syntax Highlighting 消失了——這就是訊號，告訴你「那行程式碼可能語法有問題，趕快檢查。」

此外，Vim 預設支援的 Syntax Highlighting 不僅限於程式語言。身為系統管理者，以下常見的設定檔都有專屬高亮：

- Apache、Nginx 設定檔
- SSH、sudoers、PAM 設定檔
- Git 配置、Grub 設定檔
- LDAP、PAM、Squid 設定檔

沒有內建高亮的檔案類型，也常能在社群中找到對應的 Plugin（例如 Ansible YAML 高亮）。

---

## 💡 重點摘要

- **Vim 是 VI 的改良版，在現代系統上 `vi` 命令啟動的其實就是 Vim。**
- **Vim 無所不在（Linux 預裝、Git/Sudo/Crontab 預設呼叫），不會用它等於少了一項必備技能。**
- **Vim 的命令系統是「語言式」的：Verb + Noun 結構讓你用組合代替記憶，大幅降低認知負擔。**
- **Vim 養成的鍵盤操作習慣可遷移到 Shell、Man、Less，以及支援 Vim Mode 的任何編輯器。**
- **Syntax Highlighting 的核心價值是「錯誤偵測」：某行高亮消失，往往代表語法有問題。**

---

## 🔑 關鍵字

Vim, Text Objects, Macros, Registers, Syntax Highlighting, Shell, Vim Mode
