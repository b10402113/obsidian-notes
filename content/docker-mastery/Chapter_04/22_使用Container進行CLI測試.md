# 使用 Container 進行 CLI 測試

## 📝 課程概述

本單元展示一個不同於傳統開發場景的 Container 應用：快速建立測試環境。我們將學習如何利用 Container 的輕量特性，在幾秒鐘內啟動不同 Linux 發行版，進行跨平台的工具版本測試。

## 核心觀念與實作解析

### 傳統方式的痛點

假設你是一位支援工程師，需要檢查不同 Linux 發行版上的 `curl` 版本差異：

**傳統做法**：
- 建立 VM
- 安裝作業系統（耗時數十分鐘）
- 測試完成後可能需要保留或刪除 VM

**Container 方式**：
- 秒級啟動
- 測試完即刪
- 不佔用持久資源

---

### 練習目標

在 CentOS 7 和 Ubuntu 14.04 兩個發行版中，分別檢查 `curl` 版本。

---

### 關鍵選項：`--rm`

```bash
docker container run --rm -it centos:7 bash
```

**`--rm` 的作用**：Container 停止後自動刪除，無需手動清理。

> 非常適合一次性測試場景，避免累積大量已停止的 Container。

---

### 實作步驟

**終端機一：CentOS 7**

```bash
docker container run --rm -it centos:7 bash

# 在 Container 內
yum update curl
curl --version
# 輸出：curl 7.29.0
```

**終端機二：Ubuntu 14.04**

```bash
docker container run --rm -it ubuntu:14.04 bash

# 在 Container 內
apt-get update && apt-get install -y curl
curl --version
# 輸出：curl 7.35.0
```

> 使用 iTerm2 的 `Cmd+Shift+D` 可快速分割視窗。

---

### 版本差異的啟示

| 發行版 | curl 版本 | 說明 |
|--------|----------|------|
| CentOS 7 | 7.29.0 | 企業級發行版，傾向使用較舊但穩定的版本 |
| Ubuntu 14.04 | 7.35.0 | 相對更新，即使使用較舊的 Ubuntu 版本 |

> CentOS 等企業導向發行版通常會保留較舊但經過驗證的套件版本。

---

### 離開後自動清理

當你在 Container 內輸入 `exit` 後：

```bash
docker container ls -a
# 不會看到 CentOS 或 Ubuntu Container
# --rm 已自動清理
```

---

### 實際應用場景

| 場景 | 說明 |
|------|------|
| 版本驗證 | 檢查不同 OS 的工具版本差異 |
| 快速原型 | 測試某個工具在不同環境的行為 |
| 問題排查 | 重現客戶環境中的問題 |
| 學習探索 | 體驗不同的 Linux 發行版 |

---

## 💡 重點摘要

- **`--rm` 讓 Container 在停止後自動刪除，適合一次性測試。**
- **Container 可在秒級啟動不同 Linux 發行版，取代傳統 VM 方式。**
- **企業級發行版（如 CentOS）通常使用較舊但穩定的套件版本。**
- **使用分割視窗可同時操作多個 Container 進行比較測試。**

## 🔑 關鍵字

CentOS, Ubuntu, curl, --rm, Distribution
