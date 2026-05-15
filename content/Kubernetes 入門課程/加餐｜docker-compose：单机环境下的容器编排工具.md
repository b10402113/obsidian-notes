# docker-compose：單機環境下的容器編排工具

## 📝 課程概述

本課程介紹 docker-compose 這個輕量級容器編排工具，它填補了 Docker 與 Kubernetes 之間的空白，讓開發者能在單機環境中以「宣告式」方式管理多容器應用，無需承擔 Kubernetes 的運行成本。

## 核心觀念與實作解析

### docker-compose 的起源

docker-compose 源自 2014 年一個名為 Fig 的小專案。Fig 為 Docker 引入了「容器編排」概念，使用 YAML 定義容器的啟動參數、先後順序和依賴關係，讓使用者第一次見識到「宣告式」的威力。Docker 公司收購 Fig 後，將其整合並改名為 docker-compose。

### docker-compose vs Kubernetes

雖然兩者都使用 YAML 進行容器編排，但設計理念完全不同：

| 特性 | docker-compose | Kubernetes |
|------|----------------|------------|
| 適用環境 | 單機 | 叢集 |
| 複雜度 | 低 | 高 |
| 學習曲線 | 平緩 | 陡峭 |
| 核心概念 | service | Pod、Deployment、Service 等 |

### 核心概念：service

docker-compose 的核心概念是 **service**。注意它與 Kubernetes 的 Service 雖然名稱相似，但完全不同：

- docker-compose 的 **service** 是一個容器化的應用程式
- 類似 Kubernetes 中 Pod 裡的 container
- 融合了部分 Service、Deployment 的特性

### 實作範例：私有映像檔倉庫 Registry

```yaml
services:
  registry:
    image: registry
    container_name: registry
    restart: always
    ports:
      - 5000:5000
```

啟動與管理指令：
- `docker-compose -f reg-compose.yml up -d` — 啟動應用
- `docker-compose -f reg-compose.yml ps` — 查看狀態
- `docker-compose -f reg-compose.yml down` — 停止應用

### 實作範例：WordPress 網站架設

docker-compose 的優勢在多容器編排時更明顯：

**網路識別**：每個 service 名稱同時也是該容器的唯一網路標識，類似 Kubernetes 的 Service 域名機制。
```yaml
services:
  wordpress:
    environment:
      WORDPRESS_DB_HOST: mariadb  # 直接使用 service 名稱
```

**依賴關係**：使用 `depends_on` 設定容器啟動順序。
```yaml
services:
  wordpress:
    depends_on:
      - mariadb
```

**磁碟區掛載**：使用 `volumes` 載入外部設定檔。
```yaml
services:
  nginx:
    volumes:
      - ./wp.conf:/etc/nginx/conf.d/default.conf
```

## 💡 重點摘要

- docker-compose 是單機環境下的輕量級容器編排工具，填補了 Docker 與 Kubernetes 之間的空白
- 核心概念 **service** 代表一個容器化應用，名稱同時作為網路標識
- 使用 YAML 定義，語法語義接近 Docker 命令列，學習門檻低
- `depends_on` 可設定容器啟動順序，`volumes` 可掛載外部檔案
- 適合簡單開發測試場景，GitHub 上許多專案（如 CNCF Harbor）提供 docker-compose YAML 快速搭建原型

## 🔑 關鍵字

docker-compose, service, 容器編排, YAML, depends_on