# CLI 工具容器實作練習解答

## 📝 課程概述

本單元是 Assignment 01 的解答影片，老師實作兩個 CLI 工具容器：**CMatrix**（Matrix 螢幕保護程式）與 **Apache Bench**（HTTP 效能測試工具）。透過這兩個練習，我們學習如何將 Linux 工具 Dockerize 成方便使用的容器化 CLI。

---

## 核心觀念與實作解析

### 練習一：CMatrix 容器

CMatrix 是一個終端機的 Matrix 螢幕保護程式。

#### Dockerfile

```dockerfile
FROM alpine:latest

RUN apk add --no-cache cmatrix

ENTRYPOINT ["cmatrix"]
CMD ["-C", "red"]
```

**解析：**

| 指令 | 說明 |
|------|------|
| `FROM alpine` | 使用輕量 Alpine Image |
| `apk add --no-cache` | Alpine 的套件管理器，`--no-cache` 避免快取 |
| `ENTRYPOINT ["cmatrix"]` | 固定執行 cmatrix |
| `CMD ["-C", "red"]` | 預設紅色輸出 |

#### Build 與執行

```bash
docker build -t cmatrix .
docker run -it cmatrix
```

> **注意：** CMatrix 需要終端機，必須使用 `-it` 選項。

#### 覆蓋 CMD

```bash
# 使用預設紅色
docker run -it cmatrix

# 覆蓋為彩虹模式
docker run -it cmatrix -C rainbow

# 進入 Shell 探索選項
docker run -it --entrypoint sh cmatrix
cmatrix --help
```

---

### 練習二：Apache Bench 容器

Apache Bench (ab) 是 HTTP 效能測試工具。

#### Dockerfile

```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y apache2-utils && rm -rf /var/lib/apt/lists/*

ENTRYPOINT ["ab"]
CMD ["-n", "10", "-c", "2", "https://www.google.com/"]
```

**解析：**

| 指令 | 說明 |
|------|------|
| `apt-get update` | 更新套件資料庫 |
| `apt-get install -y apache2-utils` | 安裝 ab 工具 |
| `rm -rf /var/lib/apt/lists/*` | 清理快取，減少 Image 大小 |
| `ENTRYPOINT ["ab"]` | 固定執行 ab |
| `CMD ["-n", "10", "-c", "2", "https://www.google.com/"]` | 預設測試參數 |

#### Build 與執行

```bash
docker build -t ab .
docker run ab
```

#### ab 參數說明

| 參數 | 說明 |
|------|------|
| `-n 10` | 總請求數 10 次 |
| `-c 2` | 並發數 2 |

#### 覆蓋 CMD 測試其他網站

```bash
docker run ab -n 20 -c 5 https://example.com/
```

> **注意：** ab 對 URL 格式很嚴格，結尾必須有 `/`。

---

### Alpine vs Ubuntu 套件管理

| 作業系統 | 套件管理器 | 安裝指令 | 清理快取 |
|---------|-----------|---------|---------|
| Alpine | `apk` | `apk add --no-cache <pkg>` | 自動（加 `--no-cache`） |
| Ubuntu | `apt-get` | `apt-get install -y <pkg>` | `rm -rf /var/lib/apt/lists/*` |

> Alpine 的套件安裝指令較簡潔，Ubuntu 需要額外清理步驟。

---

### JSON 語法常見錯誤

老師在影片中犯了一個常見錯誤：

```dockerfile
# ❌ 錯誤：忘記逗號或引號
CMD ["-C" "red"]

# ✅ 正確
CMD ["-C", "red"]
```

> JSON Array 每個元素都要用雙引號包住，元素間用逗號分隔。這是 Dockerfile 中最常見的語法錯誤之一。

---

### 除錯技巧

當 Container 啟動後行為不如預期：

1. **覆蓋 ENTRYPOINT 進入 Shell**
   ```bash
   docker run -it --entrypoint sh <image>
   ```

2. **在 Shell 中測試指令**
   ```bash
   cmatrix --help
   ```

3. **修正 Dockerfile 後重新 Build**

---

## 💡 重點摘要

- **CLI 工具容器設計：`ENTRYPOINT` 放程式名稱，`CMD` 放預設參數。**
- **Alpine 使用 `apk`，Ubuntu 使用 `apt-get`；清理快取的方式不同。**
- **終端機程式需要 `-it` 選項才能正常運作。**
- **JSON Array 語法要正確：雙引號、逗號分隔。**
- **覆蓋 ENTRYPOINT 進入 Shell 是重要的除錯技巧。**

---

## 🔑 關鍵字

CLI Tool, Alpine, Ubuntu, apk, apt-get, ENTRYPOINT, CMD, JSON Array
