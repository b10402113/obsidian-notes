# Swarm Stack 與生產級 Compose

## 📝 課程概述

本單元介紹 Docker Stack——這是 Swarm 版的 Docker Compose，讓你能用 YAML 檔案定義整個應用架構。我們將學習如何使用 Compose 檔案來部署 Stack，以及 `deploy` 區塊的各種生產級設定選項。

## 核心觀念與實作解析

### 什麼是 Stack？

Stack 是 Docker 1.13 新增的功能：

- **宣告式定義**：用 YAML 描述「我要的最終狀態」
- **一站式管理**：包含 Services、Networks、Volumes、Secrets
- **類似 Compose**：但專為 Swarm 生產環境設計

> 不需要安裝 `docker-compose` CLI，Docker Engine 內建支援 Stack。

---

### Stack vs. 手動建立 Service

| 方式 | 優點 | 缺點 |
|------|------|------|
| 手動 `docker service create` | 彈性高 | 命令冗長、難以追蹤 |
| Stack (Compose File) | 版本控制、可重複部署 | 需要學習 YAML 格式 |

---

### Compose 檔案的新要求

**版本要求**：必須是 `version: 3` 或以上

```yaml
version: '3'

services:
  vote:
    image: dockersamples/examplevotingapp_vote
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
        max_attempts: 3

networks:
  frontend:
  backend:
```

---

### `deploy` 區塊詳解

這是 Stack 專屬的設定區塊，定義 Swarm 相關的部署參數：

**Replicas**：

```yaml
deploy:
  replicas: 2
```

設定要運行的 Container 數量。

**Update Config**（滾動更新）：

```yaml
deploy:
  update_config:
    parallelism: 1      # 一次更新幾個
    delay: 10s          # 每批之間的延遲
    failure_action: rollback  # 失敗時回滾
```

**Restart Policy**：

```yaml
deploy:
  restart_policy:
    condition: on-failure  # 何時重啟
    delay: 5s              # 重啟延遲
    max_attempts: 3        # 最大嘗試次數
    window: 120s           # 觀察窗口
```

**Constraints**（節點限制）：

```yaml
deploy:
  placement:
    constraints:
      - node.role == manager
```

讓 Service 只運行在特定條件的 Node。

---

### Build 與 Deploy 的相容性

| 環境 | `build` 指令 | `deploy` 指令 |
|------|-------------|--------------|
| Docker Compose（本地） | ✅ 支援 | ⚠️ 忽略 |
| Docker Stack（Swarm） | ⚠️ 忽略 | ✅ 支援 |

> 你可以在同一個 Compose 檔案中同時保留 `build` 和 `deploy`，在不同環境會自動使用對應的設定。

---

### Stack 部署命令

**部署 Stack**：

```bash
docker stack deploy -c docker-compose.yml voteapp
```

- `-c`：指定 Compose 檔案
- `voteapp`：Stack 名稱

**查看 Stack 列表**：

```bash
docker stack ls
```

**查看 Stack 中的服務**：

```bash
docker stack services voteapp
```

**查看 Stack 中的 Task**：

```bash
docker stack ps voteapp
```

**移除 Stack**：

```bash
docker stack rm voteapp
```

---

### Stack 的命名規則

Stack 會自動為所有資源加上前綴：

```bash
# 原本在 Compose 中的名稱
vote

# 實際建立的名稱
voteapp_vote
```

這讓不同 Stack 之間的資源不會衝突。

---

### 更新 Stack

修改 Compose 檔案後，再次執行部署命令即可：

```bash
docker stack deploy -c docker-compose.yml voteapp
```

Swarm 會：
1. 比較現有狀態與新定義
2. 只更新有變更的部分
3. 根據 `update_config` 進行滾動更新

> **最佳實踐**：將 Compose 檔案放入 Git 版本控制，作為 Infrastructure as Code。

---

### Visualizer 工具

課程提供了一個視覺化工具：

```yaml
visualizer:
  image: dockersamples/visualizer
  ports:
    - "8080:8080"
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
  deploy:
    placement:
      constraints:
        - node.role == manager
```

- 連接 `http://<node-ip>:8080` 可看到所有 Service 在 Node 上的分佈
- 適合學習與除錯

---

## 💡 重點摘要

- **Stack 用 Compose 檔案管理 Services、Networks、Volumes、Secrets。**
- **`deploy` 區塊定義 Swarm 專屬的部署參數，如 replicas、滾動更新、重啟策略。**
- **Compose 本地會忽略 `deploy`，Stack 會忽略 `build`，兩者可共存。**
- **更新 Stack 只需重新執行 `docker stack deploy`，Swarm 會自動差異更新。**

## 🔑 關鍵字

Stack, Compose, deploy, Replica, Update
