# Elasticsearch Swarm Compose File 審查

## 📝 課程概述

本單元審查了 Jacob 提交的 Elasticsearch Stack Compose File，這是一個用於 Docker Swarm 的 Production 配置。老師針對 endpoint mode、placement constraints、volume drivers、以及 YAML templating 等進階功能提供專業建議。

---

## 核心觀念與實作解析

### 審查標的：Elasticsearch Cluster on Docker Swarm

這是一個多節點 Elasticsearch Cluster 配置，包含：

- 3 個 Master nodes（master1, master2, master3）
- 多個 Data nodes
- 使用 Traefik 作為 Proxy
- 完整的 healthcheck、configs、networks 設定

### Endpoint Mode：DNS Round Robin

```yaml
deploy:
  endpoint_mode: dnsrr
```

這是一個**非常棒的設定**，但很多人不知道！

#### 為什麼要用 DNS Round Robin？

預設情況下，Docker Swarm 會在每個 Service 前面放置一個 **Virtual IP (VIP)**：

- VIP 會再路由到實際的 Container
- 這對於多副本的 Service 是合理的
- 但對於**單副本的 Database**，VIP 只是多餘的 hop

設定 `dnsrr` 後：

- **關閉 VIP**
- DNS 直接解析到實際 Container IP
- 減少一層 NAT translation
- 降低 latency 與潛在問題

> 適用場景：單副本 Database、每個 Service 只有一個 instance 的情況。

---

### Placement Constraints 的使用

```yaml
deploy:
  placement:
    constraints:
      - node.hostname == master1
```

這樣做可以確保特定 Container 運行在特定 Node 上。

#### 優點

- 確保 Database 分散在不同實體機器
- 控制 data locality

#### 缺點

- 如果該 Node 掛掉，Container 無法被排程到其他 Node
- 需要確保這些 Node 是高可用的

#### 替代方案

**1. Placement Preferences（可用區分散）**

```yaml
deploy:
  placement:
    preferences:
      - spread: node.labels.availability_zone
```

這會讓 Swarm 嘗試將 replicas 分散到不同的 availability zone。

**2. Maximum Replicas Per Node（Docker 19.03+）**

```yaml
deploy:
  replicas: 3
  max_replicas_per_node: 1
```

這是新功能，確保同一個 Service 的多個 replicas 不會落在同一個 Node 上。

---

### Volume Driver 的考量

```yaml
volumes:
  es_data:
```

這裡使用預設的 local volume driver，意味著：

- **Data 綁定在該 Node 的磁碟上**
- 如果 Node 掛掉，Volume 無法自動遷移到其他 Node

#### 建議

對於 Production Database，應該考慮：

- **REX-Ray** 或其他 volume driver
- 支援 remote storage（如 AWS EBS、Ceph）
- Node 故障時，Volume 可以重新 attach 到其他 Node

---

### YAML Templating 減少重複

這個 Compose File 有大量重複的設定（master1, master2, master3 幾乎一樣）。

#### 解決方案：Docker Compose Templating

```yaml
x-elasticsearch: &elasticsearch-template
  image: elasticsearch:8.0
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:9200"]
  networks:
    - elasticsearch

services:
  master1:
    <<: *elasticsearch-template
    # 特定設定...
  
  master2:
    <<: *elasticsearch-template
    # 特定設定...
```

這樣可以：

- 減少 YAML 行數
- 遵循 DRY 原則
- 更容易維護

---

### 其他審查要點

#### Resource Limits

```yaml
deploy:
  resources:
    limits:
      memory: 4G
    reservations:
      memory: 1G
```

這是最佳實踐！**強烈建議所有 Production Service 都設定 resource limits**。

#### Network Mode

```yaml
ports:
  - target: 9200
    published: 9200
    mode: host
```

使用 `host` mode 讓 Container 直接存取宿主機的 NIC，跳過 overlay network，對效能有幫助。

#### Attachable Networks

```yaml
networks:
  elasticsearch:
    attachable: true
```

這讓 standalone containers 也能連接到這個 network，增加彈性。

#### Configs vs Secrets

這個 Compose File 使用了 `configs` 但沒有 `secrets`。如果有敏感資料（密碼、API keys），應該使用 Docker Secrets：

```yaml
secrets:
  elastic_password:
    external: true
```

---

## 💡 重點摘要

- **`endpoint_mode: dnsrr` 適合單副本 Database，可以關閉 VIP，減少不必要的 network hop。**
- **Placement Constraints 會讓 Service 綁定特定 Node，需考慮 Node 故障時的容錯機制。**
- **Production Database 應考慮使用 remote volume driver（如 REX-Ray），避免 data 被 lock 在特定 Node。**
- **使用 YAML templating 減少重複設定，讓 Compose File 更容易維護。**
- **務必設定 resource limits 與 reservations，確保叢集資源分配得當。**

---

## 🔑 關鍵字

Swarm, endpoint_mode, Placement, Volume Driver, Templating
