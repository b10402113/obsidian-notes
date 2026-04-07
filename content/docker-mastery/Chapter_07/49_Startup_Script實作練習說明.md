# Startup Script 實作練習

## 📝 課程概述

本單元是 Assignment 02 的練習說明。我們將建立一個 FastAPI Python 應用程式的 Dockerfile，並透過 ENTRYPOINT Script 在主程序啟動前執行初始化工作。這個練習展示了實務上常見的 Startup Script 模式。

---

## 核心觀念與實作解析

### 練習目標

建立 Dockerfile 來執行一個 FastAPI Python Web API，需要：
- 在啟動前執行初始化工作
- 建立資料目錄並設定 Volume
- 使用 ENTRYPOINT Script 處理啟動邏輯

---

### 練習目錄結構

```
assignment-02/
├── Dockerfile
├── README.md
├── app/
│   └── (FastAPI 應用程式)
├── default_data/
│   └── (預設資料檔案)
└── docker-entrypoint.sh
```

---

### 需要完成的任務

根據 README 的指示：

1. **FROM** — 使用 Python slim Image
2. **WORKDIR** — 設定為 `/app`
3. **COPY requirements.txt** — 先複製依賴檔案
4. **RUN pip install** — 安裝依賴
5. **COPY . .** — 複製所有檔案
6. **VOLUME** — 宣告 `/app/data`
7. **ENTRYPOINT** — 設定為 `docker-entrypoint.sh`
8. **CMD** — 啟動 FastAPI

---

### 關鍵設計考量

#### 為什麼先複製 requirements.txt？

```dockerfile
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
```

**原因：**
- 依賴很少變動，原始碼常變動
- 分開處理可善用 Build Cache
- 大型專案的依賴安裝可能需要數分鐘

#### VOLUME 的作用

```dockerfile
VOLUME /app/data
```

- 確保資料目錄有持久化儲存
- 即使沒有指定 Volume，Docker 也會自動建立

---

### 預期的 CMD

```dockerfile
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**為什麼 `--host 0.0.0.0`？**

> Container 內的應用程式預設常監聽 `localhost`（127.0.0.1），這表示只接受 Container 內部的連線。設定為 `0.0.0.0` 表示監聽所有介面，外部網路才能連線。

---

## 💡 重點摘要

- **先複製 requirements.txt 再 pip install，可善用 Build Cache 加速建置。**
- **`VOLUME` 確保資料持久化，即使未指定也會自動建立。**
- **`--host 0.0.0.0` 讓應用程式監聽所有介面，外部網路才能連線。**
- **ENTRYPOINT Script 用於執行啟動前的初始化工作。**
- **下一個影片將詳細解說完整的實作過程。**

---

## 🔑 關鍵字

Startup Script, ENTRYPOINT, VOLUME, FastAPI, uvicorn, Build Cache
