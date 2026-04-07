# Compose 中整合 Image Building

## 📝 課程概述

本單元學習如何在 Docker Compose 中整合 Image Building 功能。我們將看到 Compose 不只能啟動現有的 Image，還能在執行時自動建置自訂 Image，讓「本機開發環境建置」與「模擬 Production 環境」的需求能夠同時滿足。

---

## 核心觀念與實作解析

### 為什麼要在 Compose 中建置 Image？

之前我們學過用 `docker build` 手動建置 Image，但在 Compose 中整合建置有以下優勢：

1. **自動化**：團隊成員不需要記住複雜的 build 指令
2. **統一管理**：Dockerfile 和 compose.yml 放在同一個專案中
3. **一鍵啟動**：`docker-compose up` 自動處理一切

> 適用場景：你需要自訂 Image（例如複製設定檔進去），而不是直接使用官方 Image。

---

### Compose 如何處理 Image Building？

當你在 Service 中加入 `build` 設定：

1. Compose 會先檢查 Image Cache 中是否已有該 Image
2. 如果找不到，才會執行 Build
3. Build 完成後，Image 會存入本機 Cache

**重要**：Compose **不會每次都重建 Image**。如果你修改了 Dockerfile，需要明確指定：

```bash
# 強制重建 Image
docker-compose build

# 或者在啟動時重建
docker-compose up --build
```

---

### 實作範例：自訂 Nginx Proxy + Apache

**目錄結構**：
```
compose-sample-3/
├── docker-compose.yml
├── nginx.Dockerfile
├── nginx.conf
└── html/
    └── index.html
```

**docker-compose.yml**：
```yaml
version: '2'

services:
  proxy:
    build:
      context: .
      dockerfile: nginx.Dockerfile
    image: nginx-custom
    ports:
      - "80:80"

  web:
    image: httpd
    volumes:
      - ./html:/usr/local/apache2/htdocs
```

**nginx.Dockerfile**：
```dockerfile
FROM nginx:1.13

COPY nginx.conf /etc/nginx/conf.d/default.conf
```

---

### Build 設定詳解

#### 基本語法

```yaml
services:
  myservice:
    build: .
    # 簡寫：使用當前目錄的預設 Dockerfile
```

#### 完整語法

```yaml
services:
  myservice:
    build:
      context: .
      dockerfile: nginx.Dockerfile
    image: nginx-custom
```

- `context`：Build Context 的路徑
- `dockerfile`：自訂 Dockerfile 名稱（預設為 `Dockerfile`）
- `image`：建置完成後的 Image 名稱

> 如果同時有 `build` 和 `image`，`image` 的意義會改變：它變成「建置後要命名的 Image 名稱」。

---

### 執行過程解析

**第一次執行**：

```bash
docker-compose up
```

輸出：
1. 建立 Network
2. 偵測到 `nginx-custom` Image 不存在
3. 執行 `docker build`（讀取 nginx.Dockerfile）
4. 建立 Containers 並啟動

> Compose 會貼心提示：這次因為找不到 Image 所以執行了 Build。

**之後執行**：

```bash
docker-compose up
```

由於 Image 已存在，直接啟動 Containers。

---

### 開發場景：結合 Build + Bind Mount

這個範例展示了非常常見的開發模式：

```yaml
services:
  proxy:
    build: .
    # Nginx 設定檔 baked into Image（不常變動）

  web:
    image: httpd
    volumes:
      - ./html:/usr/local/apache2/htdocs
      # HTML 檔案 Bind Mount（需要常編輯）
```

**設計邏輯**：
- **不常變動的設定**：用 Build 方式放入 Image（如 nginx.conf）
- **需要即時編輯的檔案**：用 Bind Mount（如 HTML/CSS/JS）

> 這讓開發環境能盡可能模擬 Production，同時保持開發效率。

---

### 清除時移除 Image

預設的 `docker-compose down` **不會刪除建置的 Image**。如果你想要徹底清理：

```bash
# 移除「沒有自訂名稱」的 Image
docker-compose down --rmi local

# 移除「所有相關」的 Image
docker-compose down --rmi all
```

**差異說明**：

| 選項 | 行為 |
|------|------|
| `--rmi local` | 只移除 Compose 建置的 Image |
| `--rmi all` | 移除專案使用的所有 Image（包含 httpd 等官方 Image） |

> `--rmi all` 要小心使用，你可能不想每次都重新拉取 httpd Image。

---

### 省略 image 名稱的情況

如果你不指定 `image` 名稱：

```yaml
services:
  proxy:
    build: .
    # 沒有 image 設定
```

Compose 會自動產生一個名稱，格式為：
```
<project>_<service>
# 例如：compose-sample-3_proxy
```

這種方式的好處是 `docker-compose down --rmi local` 能更精確地清除。

---

### Build Arguments

如果 Dockerfile 使用 `ARG`：

```dockerfile
ARG NODE_VERSION
FROM node:${NODE_VERSION}
```

可以在 Compose 中傳入：

```yaml
services:
  app:
    build:
      context: .
      args:
        NODE_VERSION: 14
```

> Build Arguments 只在 Build 階段有效，不會保留在最終 Image 中。

---

## 💡 重點摘要

- **Compose 能自動建置 Image，只需在 Service 中加入 `build` 設定。**
- **Compose 不會每次都重建，修改 Dockerfile 後需用 `--build` 或 `docker-compose build`。**
- **同時有 `build` 和 `image` 時，`image` 變成「建置後的 Image 名稱」。**
- **`docker-compose down --rmi local` 能清除建置的 Image，保持環境乾淨。**
- **Build + Bind Mount 組合是常見的開發模式：設定檔 baked in、原始碼即時編輯。**

---

## 🔑 關鍵字

build, context, dockerfile, Build Arguments, rmi
