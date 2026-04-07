# Docker 快速實作體驗

## 📝 課程概述

在深入安裝 Docker 之前，本單元先讓學員透過瀏覽器體驗 Docker 的基本操作。我們將使用 Docker 官方提供的免費雲端環境 Play with Docker，實際執行 Container、理解 Docker 的 Client-Server 架構，並觀察 Port 映射與隔離機制。

---

## 核心觀念與實作解析

### 環境準備：Play with Docker

在本地安裝 Docker 之前，老師建議先使用 **Play with Docker** 這個免費的雲端練習環境。

#### 註冊 Docker 帳號

1. 前往 **hub.docker.com** 註冊免費帳號
2. 只需要 Email 與 Docker ID（使用者名稱）
3. **重要：** 登入時使用 Docker ID 與密碼，而非 Email

> Play with Docker 需要登入是為了防止濫用免費資源。

#### 啟動 Play with Docker 環境

1. 前往 `labs.play-with-docker.com`
2. 使用 Docker 帳號登入
3. 點擊 **Start** 按鈕（綠色）
4. 點擊 **Add new instance** 建立 Docker 環境

**環境限制：** 每個 Instance 最長使用 4 小時，時間到後會自動銷毀。

---

### Docker 的 Client-Server 架構

進入環境後，執行第一個指令：

```bash
docker version
```

這個指令會回傳兩個區塊的資訊：
- **Client**：你輸入指令的 CLI 工具
- **Server（Engine）**：背景執行的 Docker Engine

#### Docker 通訊方式

Client 與 Engine 之間透過以下協定通訊：

| 協定 | 使用場景 |
|------|---------|
| **Socket** | 預設方式，本地通訊 |
| **TCP** | 遠端連線 |
| **TLS** | 加密的遠端連線 |
| **SSH Tunnel** | 透過 SSH 連線 |

> Docker CLI（`docker` 指令）只是 Client，實際執行 Container 的是背景的 Docker Engine。

---

### 實作：執行第一個 Container

使用以下指令啟動一個 Apache 網頁伺服器：

```bash
docker run -d -p 8800:80 httpd
```

#### 指令解析

| 參數 | 說明 |
|------|------|
| `docker run` | 從 Image 建立 Container 並執行 |
| `-d` | **Detach** 模式，Container 在背景執行 |
| `-p 8800:80` | Port 映射，Host 的 8800 → Container 的 80 |
| `httpd` | Image 名稱（Apache 的 daemon） |

#### 執行過程發生的事

當你執行 `docker run` 時，Docker 會自動：

1. **下載 Image Layers** — 如果本地沒有 `httpd` Image
2. **建立虛擬網路介面** — 給 Container 專用的 IP
3. **準備檔案系統** — 將 Image 中的檔案載入 Container
4. **啟動程序** — 在獨立的 Namespace 中執行 `httpd`

---

### 驗證 Container 執行狀態

#### 方法一：使用 curl 測試

```bash
curl localhost:8800
```

若 Container 正常運作，會回傳 Apache 的預設網頁內容。

#### 方法二：查看執行中的 Container

```bash
docker ps
```

這個指令會列出所有執行中的 Container，包括：
- Container ID
- Image 名稱
- 執行的指令
- 建立時間
- 狀態
- Port 映射

---

### 執行多個 Container

Docker 的隔離特性讓我們能同時執行多個相同或不同版本的應用程式。

#### 啟動第二個 Apache Container

```bash
docker run -d -p 8801:80 httpd
```

**關鍵差異：** Host Port 改為 8801（因為 8800 已被第一個 Container 使用）。

> 如果嘗試使用已被佔用的 Port，會得到錯誤：`port is already in use`

#### 隔離機制說明

```
┌─────────────────────────────────────────────────┐
│                    Linux Host                    │
│                                                  │
│  ┌─────────────────┐   ┌─────────────────┐      │
│  │   Container 1   │   │   Container 2   │      │
│  │                 │   │                 │      │
│  │  IP: 172.x.x.1  │   │  IP: 172.x.x.2  │      │
│  │  Port: 80       │   │  Port: 80       │      │
│  │  httpd process  │   │  httpd process  │      │
│  └─────────────────┘   └─────────────────┘      │
│         ▲                      ▲                 │
│         │                      │                 │
│    Port 8800              Port 8801              │
│         │                      │                 │
└─────────┼──────────────────────┼─────────────────┘
          │                      │
      外部流量                外部流量
```

**每個 Container 都有：**
- 獨立的 Private IP（在 Docker 虛擬網路內）
- 自己的 Port 80（Container 內部）
- 完全隔離的檔案系統與程序空間

---

### Docker 底層的 Linux 技術

老師最後提到，Container 的「魔術」其實來自 Linux 的核心功能：

| 技術 | 功能 |
|------|------|
| **Namespaces** | 隔離程序視野（看不見 Host 其他程序） |
| **Cgroups** | 資源限制（CPU、記憶體等） |
| **Veth** | 虛擬網路介面 |
| **IPTABLES** | 防火牆與 Port 轉發 |
| **Union Mounts** | 多層檔案系統疊加 |

當你進入 Container 內部時，看起來就像一個普通的 Linux 環境，只是檔案更少。

---

## 💡 重點摘要

- **Docker 採用 Client-Server 架構：CLI 是 Client，Docker Engine 是 Server，兩者透過 Socket/TCP/SSH 通訊。**
- **`docker run -d -p <host_port>:<container_port> <image>` 是啟動 Container 的基本指令，`-d` 為背景執行，`-p` 為 Port 映射。**
- **每個 Container 獲得獨立的 IP、檔案系統與程序空間，因此可同時執行多個相同應用程式的不同版本。**
- **Port 映射時，Host Port 不可重複，但 Container 內部 Port 可以相同（因為 IP 不同）。**
- **Play with Docker 是免費的雲端練習環境，每個 Instance 最長可使用 4 小時。**

---

## 🔑 關鍵字

Docker CLI, Docker Engine, Container, Image, Port Mapping, Namespace, Play with Docker, httpd
