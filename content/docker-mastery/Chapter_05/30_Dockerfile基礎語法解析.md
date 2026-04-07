# Dockerfile 基礎語法解析

## 📝 課程概述

本單元正式進入 Dockerfile 的世界。Dockerfile 是建立 Container Image 的「配方」，透過一系列指令（Stanza）定義 Image 的組成內容。我們將逐一解析 Dockerfile 的核心指令，理解每個指令的作用與最佳實踐。

---

## 核心觀念與實作解析

### Dockerfile 是什麼？

> **Dockerfile 是建立 Image 的配方，不是 Shell Script。**

Dockerfile 使用 Docker 專屬的語法，預設檔名為 `Dockerfile`（大寫 D）。若需使用不同檔名，可透過 `-f` 參數指定。

```bash
docker build -f MyDockerfile .
```

---

### 核心指令解析

#### FROM — 指定 Base Image

每個 Dockerfile 都必須以 `FROM` 開頭，指定基礎 Image。

```dockerfile
FROM debian:jessie
```

**為什麼要使用 Base Image？**
- 節省時間：不需要從頭安裝 OS
- 安全性：官方 Image 有持續更新安全補丁
- 套件管理：可使用該發行版的 Package Manager

**常見 Base Image 選擇：**
- `debian` / `ubuntu` — 功能完整，適合需要多套件的場景
- `alpine` — 極小（~5MB），適合追求最小化

---

#### ENV — 設定環境變數

```dockerfile
ENV NGINX_VERSION 1.10.1
```

環境變數在 Container 中非常重要：
- 後續指令可使用此變數
- Container 執行時也可存取

---

#### RUN — 執行指令

`RUN` 會在 Build 過程中執行指令：

```dockerfile
RUN apt-get update && apt-get install -y \
    nginx=${NGINX_VERSION} \
    && rm -rf /var/lib/apt/lists/*
```

**為什麼用 `&&` 串接指令？**

每個 Dockerfile 指令都會建立一個 Layer。將多個指令用 `&&` 串接，可以：

1. **減少 Layer 數量**
2. **節省 Image 大小**
3. **這是 Dockerfile 的常見最佳實踐**

> 清理指令（如 `rm -rf /var/lib/apt/lists/*`）應與安裝指令放在同一個 `RUN`，確保暫存檔不會被保留在 Layer 中。

---

#### 關於 Logging 的最佳實踐

```dockerfile
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log
```

**為什麼要將 Log 導向 stdout/stderr？**

- Container 內不需要 syslog daemon
- Docker Engine 會自動收集 stdout/stderr
- 可透過 Docker Logging Driver 統一管理
- 避免在 Container 內處理 Log 檔案的複雜性

> **最佳實踐：Container 內的應用程式應將 Log 輸出到 stdout/stderr，讓 Docker 處理後續。**

---

#### EXPOSE — 宣告 Port

```dockerfile
EXPOSE 80 443
```

**重要觀念：**
- `EXPOSE` 只是 **Metadata**，不會實際開啟 Port
- 真正開啟 Port 需要在 `docker run` 時使用 `-p` 參數
- `EXPOSE` 告訴使用者這個 Image 預期開放的 Port

---

#### CMD — 預設執行指令

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
```

`CMD` 定義 Container 啟動時執行的預設指令。這是每個 Container 啟動時會執行的命令。

---

### Build Image

使用 `docker build` 建立 Image：

```bash
docker image build -t custom-nginx .
```

**參數說明：**
- `-t` — 指定 Image 名稱（Tag）
- `.` — 指定 Build Context 目錄

---

### Build Cache 機制

Docker 會快取每個 Layer 的建置結果。當 Dockerfile 未變更時，Build 會非常快速。

#### 變更順序的重要性

```dockerfile
# 如果 Step 5 變更了...
# Step 5 之後的所有 Layer 都需要重建
FROM debian          # Step 1 - 使用 cache
ENV NGINX_VERSION... # Step 2 - 使用 cache
RUN apt-get update...# Step 3 - 使用 cache
RUN ln -sf /dev/...  # Step 4 - 使用 cache
EXPOSE 80 8080       # Step 5 - 變更！重建此層
CMD ["nginx"...]     # Step 6 - 重建此層
```

**最佳實踐：**

> **將變動頻率低的指令放在 Dockerfile 前方，變動頻率高的放在後方。**

這樣可以最大化 Build Cache 的效益。例如：
- 套件安裝 → 放前面
- 原始碼複製 → 放後面

---

## 💡 重點摘要

- **Dockerfile 是建立 Image 的配方，每個指令（Stanza）會建立一個 Layer。**
- **`FROM` 是必要指令，指定 Base Image；`CMD` 定義 Container 啟動時的預設指令。**
- **`RUN` 指令用 `&&` 串接可減少 Layer 數量，是常見最佳實踐。**
- **Log 應輸出到 stdout/stderr，讓 Docker Engine 統一處理。**
- **Build Cache 機制：變更某行後，該行及之後所有 Layer 都需重建，故應將變動少的指令放前面。**

---

## 🔑 關鍵字

Dockerfile, FROM, RUN, CMD, ENV, EXPOSE, Build Cache, Layer
