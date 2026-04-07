# Routing Mesh 路由網格機制

## 📝 課程概述

本單元深入探討 Swarm 的 Routing Mesh 機制。這是讓任何 Node 都能接收外部流量並轉送到正確 Container 的關鍵技術。我們將理解 VIP（Virtual IP）與外部流量路由的運作原理，以及其限制與解決方案。

## 核心觀念與實作解析

### 什麼是 Routing Mesh？

Routing Mesh 是 Swarm 的**入口（Ingress）網路**，負責：

- 讓所有 Node 監聽已發布的 Port
- 將外部流量負載平衡到正確的 Container
- 使用 Linux Kernel 的 **IPVS** 技術實現

> IPVS 是 Linux Kernel 長期以來就有的功能，不是 Docker 發明的新技術。

---

### 兩種 Routing 場景

**場景一：Service 對 Service 通訊（內部）**

當一個 Service 需要連接另一個 Service 時：

```
┌─────────────────────────────────────────────────┐
│                Overlay Network                  │
│                                                 │
│   Frontend Service        Backend Service       │
│   ┌─────────┐             ┌─────────┐          │
│   │   Web   │────────────►│   VIP   │          │
│   └─────────┘             └────┬────┘          │
│                                │               │
│                    ┌───────────┼───────────┐   │
│                    ▼           ▼           ▼   │
│               ┌───────┐  ┌───────┐  ┌───────┐ │
│               │ Task1 │  │ Task2 │  │ Task3 │ │
│               └───────┘  └───────┘  └───────┘ │
└─────────────────────────────────────────────────┘
```

**關鍵概念：VIP（Virtual IP）**

- Swarm 為每個 Service 建立一個 VIP
- VIP 負責負載平衡到所有 Task
- **不是 DNS Round Robin**（避免 DNS 快取問題）

> VIP 類似於硬體負載平衡器的運作方式，比 DNS Round Robin 更可靠。

---

**場景二：外部流量進入（Ingress）**

```
                    外部流量
                        │
         ┌──────────────┼──────────────┐
         ▼              ▼              ▼
    ┌─────────┐    ┌─────────┐    ┌─────────┐
    │ Node 1  │    │ Node 2  │    │ Node 3  │
    │ :80     │    │ :80     │    │ :80     │
    └────┬────┘    └────┬────┘    └────┬────┘
         │              │              │
         └──────────────┼──────────────┘
                        │
                  ┌─────▼─────┐
                  │ Container │
                  │ (可能在任 │
                  │  何 Node)  │
                  └───────────┘
```

**運作機制**：
1. 外部請求可以到達任何 Node
2. 該 Node 的 Load Balancer 接收流量
3. 決定將流量轉送到哪個 Container（可能在同一 Node 或其他 Node）
4. 若在其他 Node，透過 Overlay Network 轉送

---

### 實作驗證：Elasticsearch

**建立 3 個 Replica 的 Service**：

```bash
docker service create --name search --replicas 3 -p 9200:9200 elasticsearch:2
```

**查看 Task 分佈**：

```bash
docker service ps search
```

會發現 3 個 Task 分散在不同 Node。

**測試 Load Balancing**：

```bash
curl localhost:9200
```

多次執行會看到不同的 Elasticsearch 節點名稱（如 `Patch`、`Jane Foster` 等），證明 VIP 正在負載平衡。

---

### Routing Mesh 的限制

| 限制 | 說明 |
|------|------|
| **無狀態負載平衡** | 每次請求可能到不同 Container，Session Cookie 不適用 |
| **Layer 4 負載平衡** | 只能基於 IP + Port，無法根據網域名稱路由 |
| **WebSocket 限制** | 需要持久連線的協定不適用 |

> 如果應用需要 Session 親和性（Session Affinity），需要額外處理。

---

### 解決方案

**方案一：Nginx / HAProxy**

在 Routing Mesh 前面部署一個狀態感知的 Load Balancer：
- 支援 Session Affinity
- Layer 7 路由（根據網域名稱）
- 快取等功能

**方案二：Docker Enterprise (UCP)**

付費的 Docker Enterprise 版本內建：
- Layer 7 Web Proxy
- 只需在 Service 設定 DNS 名稱即可

---

## 💡 重點摘要

- **Routing Mesh 讓所有 Node 都能接收外部流量，並負載平衡到正確 Container。**
- **內部通訊使用 VIP（Virtual IP），比 DNS Round Robin 更可靠。**
- **預設是無狀態的 Layer 4 負載平衡，不適合需要 Session 親和性的應用。**
- **複雜場景可搭配 Nginx、HAProxy 或 Docker Enterprise 解決。**

## 🔑 關鍵字

Routing Mesh, VIP, IPVS, Load Balancer, Ingress
