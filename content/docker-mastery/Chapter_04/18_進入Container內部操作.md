# 進入 Container 內部操作

## 📝 課程概述

本單元將探討如何進入 Container 內部進行即時操作。我們不需要在 Container 內安裝 SSH Server，而是透過 Docker 原生的 `-it` 選項與 `exec` 命令，直接取得 Container 內的 Shell 環境。同時也會介紹輕量級 Linux 發行版 Alpine 的特性。

## 核心觀念與實作解析

### 為什麼不需要 SSH？

很多初學者會問：「如何 SSH 進 Container？」實際上我們**不需要在 Container 內安裝 SSH Server**。Docker 提供了更直接的方式讓我們進入 Container 操作，既安全又方便。

---

### 使用 `docker container run -it` 取得互動式 Shell

當我們想要在啟動 Container 時直接進入 Shell，可以使用 `-it` 選項：

```bash
docker container run -it nginx bash
```

**`-it` 的組成含義**：
- `-i`（interactive）：保持 Session 開啟，讓我們能持續輸入命令
- `-t`（tty）：提供虛擬終端機（pseudo-TTY），類似 SSH 的提示符介面

> 這兩個選項通常一起使用，缺一不可。

執行後，你會發現提示符變成 `root@<container_id>`，這表示你已經進入 Container 內部。此時：
- 你是 Container 內的 root 用戶，**不是** Host 的 root
- 可以執行任何管理操作：修改配置、安裝套件等

**重要觀念**：當你輸入 `exit` 離開 Shell 時，**Container 也會停止**。因為 Container 只在啟動命令運行時才會持續存在。

---

### 使用完整 Linux 發行版 Container

我們也可以使用完整的 Linux 發行版 Image 來建立一個臨時的操作環境：

```bash
docker container run -it ubuntu bash
```

**Container 內的 Linux 發行版 vs. 標準安裝**：

| 項目 | Container 版本 | 標準安裝（VM/實體機） |
|------|---------------|---------------------|
| 預設軟體 | 非常精簡 | 較多預設軟體 |
| 套件管理 | 可用 apt/apk 等安裝 | 已有完整工具 |
| 用途 | 適合客製化環境 | 適合完整系統 |

你可以使用 `apt-get install -y curl` 安裝需要的套件。**注意**：安裝的軟體只存在於該 Container 中，若建立新的 Container 則不會包含。

---

### 重新進入已存在的 Container

若要重新啟動並進入已存在的 Container：

```bash
docker container start -ai <container_name>
```

- `-a`（attach）：連接到 Container 的標準輸出/輸入
- `-i`（interactive）：保持 Session 開啟

---

### 使用 `docker container exec` 進入運行中的 Container

對於**已經在運行**的 Container（例如 MySQL、Nginx），我們使用 `exec` 命令：

```bash
docker container exec -it <container_name> bash
```

**`run` 與 `exec` 的關鍵差異**：

| 命令 | 用途 | 影響 |
|------|------|------|
| `docker container run -it` | 啟動新 Container 並進入 | 建立新 Container |
| `docker container exec -it` | 在運行中的 Container 執行額外程序 | 不影響原有程序 |

當你 `exit` 離開 `exec` 的 Shell 時，**原本的主程序（如 MySQL Daemon）仍會繼續運行**，Container 不會停止。

---

### Alpine Linux — 輕量級的選擇

Alpine 是一個專為容器設計的輕量級 Linux 發行版：

- **Image 大小僅約 5MB**
- 使用 `apk` 作為套件管理器
- 預設不包含 `bash`，只有 `sh`

```bash
docker container run -it alpine sh
```

> 如果嘗試執行 `bash` 會得到 "executable file not found" 錯誤，因為 Alpine 預設沒有 bash。

若需要 bash，可用 `apk add bash` 安裝。

---

## 💡 重點摘要

- **不需要在 Container 內安裝 SSH，使用 `docker container run -it` 或 `exec -it` 即可直接取得 Shell。**
- **`-it` 是 `-i`（保持連線）與 `-t`（虛擬終端機）的組合，兩者缺一不可。**
- **`run -it` 啟動新 Container；`exec -it` 在運行中的 Container 執行額外程序。**
- **Alpine Linux 僅 5MB，是容器化的理想選擇，但需注意預設沒有 bash。**

## 🔑 關鍵字

Container, Shell, exec, Alpine, bash
