# Docker 網路概念解析

## 📝 課程概述

本單元將深入探討 Docker 的網路架構概念。我們將理解虛擬網路如何運作、Container 之間如何通訊，以及 `-p` 選項背後的原理。掌握這些概念後，在實際操作 CLI 時會更加得心應手。

## 核心觀念與實作解析
![[Pasted image 20260414083811.png|700]]

### Docker 的設計哲學：Batteries Included but Removable

Docker 的網路設計遵循一個核心哲學：**「Batteries Included but Removable」**：

- **Batteries Included**：預設值即可運作，開箱即用
- **But Removable**：所有設定都可以調整與客製化

> 這意味著你可以用預設設定快速開始，但當需要進階配置時，也有充分的彈性。

---

### Container 的預設網路行為

當你啟動一個 Container 時，Docker 在背景做了以下事情：

1. 將 Container 連接到預設的 **bridge** 網路
2. 透過 **NAT Firewall** 讓 Container 能對外連線
3. Container 使用與 Host 不同的 IP 網段

**查看 Container 的 IP 位址**：

```bash
docker container inspect --format '{{.NetworkSettings.IPAddress}}' <container_name>
```

> 你會發現 Container 的 IP 與 Host 的 IP 在不同的網段。

---

### 虛擬網路架構圖解

讓我們用一個架構圖來理解 Docker 網路：

```
┌─────────────────────────────────────────────────────────────┐
│                     Host Operating System                   │
│  ┌─────────────────┐     ┌─────────────────────────────┐    │
│  │  Ethernet/ NIC  │     │      Virtual Network        │    │
│  │   (Host IP)     │     │       (bridge/docker0)      │    │
│  │                 │     │  ┌───────┐    ┌───────┐     │    │
│  │  Port 80 ←──────┼─────┼──│  C1   │◄──►│  C2   │     │    │
│  │  Port 8080 ←────┼─────┼──│ Nginx │    │ MySQL │     │    │
│  │                 │     │  └───────┘    └───────┘     │    │
│  └─────────────────┘     └─────────────────────────────┘    │
│            │                                                │
│            ▼                                                │
│     [NAT Firewall]                                          │
│            │                                                │
└────────────┼────────────────────────────────────────────────┘
             │
        Physical Network
```

**關鍵要點**：
- Host 上有防火牆，預設阻擋所有來自外部網路的連入流量
- Container 出去的流量會經過 NAT 轉換
- 同一虛擬網路內的 Container 可互相通訊（無需 `-p`）

---

### `-p` 選項的作用

```bash
docker container run -p 80:80 nginx
```

這個命令的運作方式：

1. **左側（80）**：Host 上開啟的 Port
2. **右側（80）**：Container 內的 Port
3. 任何進入 Host Port 80 的流量，都會被轉送到 Container 的 Port 80

> **重要限制**：一個 Host Port 只能對應到一個 Container。不能有兩個 Container 同時監聽 Host 的同一個 Port。

---

### 多虛擬網路的應用場景

你可以建立多個虛擬網路來隔離不同的應用：

```
┌─────────────────────────────────────────────────────────────┐
│                     Host Operating System                    │
│  ┌───────────────────────────────────────┐                  │
│  │     Virtual Network: my_app            │                  │
│  │  ┌─────────┐    ┌─────────┐           │                  │
│  │  │  MySQL  │◄──►│ Apache  │           │                  │
│  │  │ :3306   │    │ :80     │           │                  │
│  │  └─────────┘    └─────────┘           │                  │
│  └───────────────────────────────────────┘                  │
│  ┌───────────────────────────────────────┐                  │
│  │     Virtual Network: bridge (default)  │                  │
│  │  ┌─────────┐    ┌─────────┐           │                  │
│  │  │  C1     │◄──►│  C2     │           │                  │
│  │  └─────────┘    └─────────┘           │                  │
│  └───────────────────────────────────────┘                  │
└─────────────────────────────────────────────────────────────┘
```

**設計原則**：
- 相關聯的 Container 放在同一個虛擬網路
- 不同應用使用不同虛擬網路，達到隔離效果
- 不需要對外開放的服務（如資料庫）不要使用 `-p`

---

### 可調整的網路設定

Docker 允許你調整多項網路設定：

| 設定項目 | 說明 |
|---------|------|
| 建立多個虛擬網路 | 一個應用一個網路 |
| Container 連接多個網路 | 類似實體機器有多張網卡 |
| Container 不連接任何網路 | 完全隔離 |
| 使用 `--net=host` | 直接使用 Host 網路（會失去部分隔離優勢） |

---

### 查看 Container Port 映射

快速查看某個 Container 的 Port 映射：

```bash
docker container port <container_name>
```

輸出範例：
```
80/tcp -> 0.0.0.0:80
```

---

## 💡 重點摘要

- **Docker 網路預設值可即用，但所有設定都可以客製化（Batteries Included but Removable）。**
- **同一虛擬網路內的 Container 可互相通訊，不需要 `-p`。**
- **`-p` 是將 Host Port 暴露給外部網路，一個 Host Port 只能對應一個 Container。**
- **建立多個虛擬網路可隔離不同應用，提升安全性。**

## 🔑 關鍵字

Network, Bridge, NAT, Port, Firewall
