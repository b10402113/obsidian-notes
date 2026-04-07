# Startup Script 實作練習解答

## 📝 課程概述

本單元是 Assignment 02 的完整解答。老師實作 FastAPI 應用程式的 Dockerfile 與 ENTRYPOINT Script，並詳細解說 Script 中的各種啟動邏輯，包括資料目錄初始化、Secret 檔案處理、環境變數檢查等實務技巧。

---

## 核心觀念與實作解析

### Dockerfile 實作

```dockerfile
FROM python:slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

VOLUME /app/data

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

### Build 與執行

```bash
docker build -t fastapi .
docker run -p 8000:8000 fastapi
```

瀏覽器開啟 `localhost:8000` 即可看到 API 回應。

---

### ENTRYPOINT Script 詳解

#### 基本結構

```bash
#!/bin/sh
set -e

# 初始化邏輯...

exec "$@"
```

**`set -e` 的作用：**
- 任何指令失敗時立即結束 Script
- 確保初始化失敗時 Container 不會啟動

**`exec "$@"` 的作用：**
- 將執行權交給 CMD 指定的指令
- 讓主程序成為 PID 1

---

#### 功能一：初始化資料目錄

```bash
if [ ! -d "/app/data" ]; then
    mkdir -p /app/data
fi

if [ -d "/app/default_data" ]; then
    cp -r /app/default_data/* /app/data/
fi
```

**用途：**
- 檢查資料目錄是否存在
- 複製預設資料到資料目錄（Seeding）

---

#### 功能二：Secret 檔案轉環境變數

```bash
for secret_file in /run/secrets/*; do
    if [ -f "$secret_file" ]; then
        secret_name=$(basename "$secret_file")
        export "$secret_name=$(cat "$secret_file")"
    fi
done
```

**用途：**
- Docker/Kubernetes Secrets 可能以檔案形式注入
- 許多應用程式期望環境變數
- Script 將檔案內容轉為環境變數

---

#### 功能三：環境變數寫入設定檔

```bash
envsubst < /app/config.template > /app/config.json
```

**`envsubst` 的作用：**
- 將設定檔中的環境變數替換為實際值
- 適合需要設定檔的舊式應用程式

---

#### 功能四：檢查必要環境變數

```bash
if [ -z "$PORT" ]; then
    echo "Error: PORT environment variable is not set"
    exit 1
fi
```

**用途：**
- 在啟動前驗證必要設定
- 及早發現問題，避免啟動後才失敗

---

### 除錯技巧

當 ENTRYPOINT Script 有問題時：

1. **覆蓋 ENTRYPOINT 進入 Shell**
   ```bash
   docker run -it --entrypoint sh fastapi
   ```

2. **在 Container 內測試 Script**
   ```bash
   sh docker-entrypoint.sh
   ```

3. **查看錯誤碼**
   ```bash
   docker inspect <container_id> | grep ExitCode
   ```

---

### 什麼情況需要 ENTRYPOINT Script？

**需要：**
- 舊式應用程式（Monolith、Pre-container）
- 需要 Secret 檔案轉環境變數
- 需要動態產生設定檔
- 需要初始化資料目錄

**不需要：**
- 現代化應用程式（12-Factor App）
- 應用程式自己處理初始化邏輯
- 設計為 Container-friendly 的應用

> **現代應用程式應該自己處理初始化邏輯**，不需要依賴 ENTRYPOINT Script。這是「12-Factor App」的設計哲學。

---

## 💡 重點摘要

- **`set -e` 確保 Script 任一指令失敗時立即結束，避免 Container 在錯誤狀態啟動。**
- **`exec "$@"` 將執行權交給 CMD，確保主程序成為 PID 1。**
- **ENTRYPOINT Script 可處理：資料初始化、Secret 轉環境變數、設定檔產生、環境變數檢查。**
- **現代應用程式（12-Factor App）應自己處理初始化，減少對 ENTRYPOINT Script 的依賴。**
- **除錯時可覆蓋 ENTRYPOINT 進入 Shell，直接測試 Script。**

---

## 🔑 關鍵字

ENTRYPOINT Script, set -e, exec, envsubst, Secret, 12-Factor App
