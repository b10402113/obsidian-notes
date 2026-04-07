# 使用 Overlay 網路跨節點擴展

## 📝 課程概述

本單元介紹 Swarm 中最重要的網路概念：Overlay Network。這是讓跨節點的 Container 能夠像在同一個區域網路般通訊的關鍵技術。我們將學習如何建立 Overlay 網路，並透過實際部署 Drupal 與 PostgreSQL 來驗證跨節點通訊。

## 核心觀念與實作解析

### 什麼是 Overlay Network？

Overlay Network 是 Swarm 專用的網路 Driver，其核心特性：

- **跨節點通訊**：讓不同 Node 上的 Container 能夠互相通訊，就像在同一個 VLAN
- **Swarm 層級**：網路層級跨越整個 Swarm，而非單一 Node
- **僅限內部通訊**：專用於 Swarm 內部服務間的通訊

> 這就像為 Swarm 建立了一個虛擬的「跨主機交換機」，所有連接的 Container 都在同一個廣播域。

---

### 建立 Overlay Network

```bash
docker network create --driver overlay mydrupal
```

**查看網路列表**：

```bash
docker network ls
```

輸出會顯示：
- `overlay`：我們剛建立的網路
- `ingress`：預設的入口網路（用於 Routing Mesh）
- `docker_gwbridge`：對外通訊用的閘道網路

---

### 實作：Drupal + PostgreSQL 跨節點部署

**步驟一：建立資料庫 Service**

```bash
docker service create --name psql --network mydrupal -e POSTGRES_PASSWORD=mypassword postgres
```

**步驟二：建立 Drupal Service**

```bash
docker service create --name drupal --network mydrupal -p 80:80 drupal
```

**驗證 Service 狀態**：

```bash
# 查看 Service 列表
docker service ls

# 查看 Service 運行在哪些 Node
docker service ps psql
docker service ps drupal
```

> 你可能會發現 psql 在 Node 1，而 drupal 在 Node 2——它們在不同節點上！

---

### 跨節點 DNS 解析

**關鍵發現**：Drupal 如何知道如何連接 PostgreSQL？

答案就是** Service 名稱作為 DNS 名稱**：

- 當 Drupal 需要連接資料庫時，只需使用 Service 名稱 `psql`
- Overlay Network 自動提供 DNS 解析
- 無需知道對方在哪個 Node 或實際 IP

**Drupal 安裝設定**：

| 設定項目 | 值 |
|---------|---|
| Database name | postgres |
| Database username | postgres |
| Database password | mypassword |
| Host | **psql**（Service 名稱） |

> 這與 Docker Compose 的行為一致——使用 Service 名稱作為主機名稱。

---

### Overlay Network 的加密選項

```bash
docker network create --driver overlay --opt encrypted mynet
```

- 使用 **IPSec** 加密節點間的所有流量
- 預設關閉（考量效能）
- 適用於需要額外安全層的場景

---

### 多網路架構設計

你可以為不同層級的服務建立不同的 Overlay Network：

```
┌─────────────────────────────────────────────────────┐
│                    Swarm Cluster                    │
│  ┌─────────────────┐      ┌─────────────────────┐  │
│  │  Frontend Net   │      │    Backend Net      │  │
│  │  ┌───────┐      │      │    ┌───────┐       │  │
│  │  │ Web   │◄─────┼──────┼───►│  API  │       │  │
│  │  └───────┘      │      │    └───────┘       │  │
│  └─────────────────┘      │         │          │  │
│                           │    ┌────▼────┐     │  │
│                           │    │   DB    │     │  │
│                           │    └─────────┘     │  │
│                           └─────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

**設計原則**：
- Web 層只連接 Frontend Network
- API 層連接兩個網路（前後端橋接）
- DB 層只連接 Backend Network（隔離保護）

---

## 💡 重點摘要

- **Overlay Network 讓跨節點 Container 能像在同一個區域網路般通訊。**
- **Service 名稱自動成為 DNS 名稱，無需手動設定 IP。**
- **可使用 `--opt encrypted` 啟用 IPSec 加密。**
- **多網路架構可實現分層安全隔離。**

## 🔑 關鍵字

Overlay, Swarm, Service, DNS, Network
