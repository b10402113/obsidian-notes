# Docker 生產環境實戰：最佳實踐與決策指南

## 📝 課程概述

本單元是講師在 DockerCon 2017 的演講記錄，涵蓋從硬體決策到 OS 選擇、Dockerfile 撰寫、Swarm 架構設計等完整範疇。內容聚焦於進入生產環境前需要做出的關鍵決策，以及如何避免常見的 Anti-pattern。

## 核心觀念與實作解析

### 限制同時創新的範圍

這是「範疇蔓延」(Scope Creep) 的問題。許多專案在第一次容器化時，就想把過去 15-20 年建立的 VM 基礎架構功能全部複製過來。

**第一個容器專案可以暫緩的需求**：

| 需求 | 建議 |
|------|------|
| CI/CD Pipeline | 若現有系統已支援容器，可延後重建 |
| 動態效能擴展 | 需先累積經驗再自動化 |
| 持久化資料庫 | 先從無狀態應用開始 |
| Twelve-Factor 完美實踐 | 視為理想目標，非必要條件 |

> Twelve-Factor 應該視為「地平線」而非「終點」，逐步學習分散式運算最佳實踐即可。

### Dockerfile 品質優先

Dockerfile 是新的建置文件，需要：

1. **豐富的註解與文件** — 這些在 Build 前會被移除，不影響 Image 大小
2. **版本固定** — 永遠不要使用 `latest` Tag
3. **環境變數集中管理** — 在 Dockerfile 開頭宣告版本號

```dockerfile
# 在開頭宣告版本，一目了然
ENV NODE_VERSION=14.17.0
ENV APP_VERSION=1.2.3

FROM node:${NODE_VERSION}
# ...
```

**Image 大小不是首要問題**。一個 500MB 的 Image 在每台主機只儲存一次，即使運行五個 Container 也是如此。先關注 Dockerfile 品質，再考慮改用 Alpine 等精簡 Image。

### 常見 Dockerfile Anti-pattern

#### 1. 忘記設定 Volume

```dockerfile
# 確保持久化資料有正確的 Volume
VOLUME /var/log/app
VOLUME /data/uploads
```

#### 2. 使用 latest Tag

```dockerfile
# ❌ 錯誤
FROM node:latest

# ✅ 正確
FROM node:14.17.0
```

#### 3. 不同環境建立不同 Image

> Anti-pattern：為 dev/staging/prod 環境建立不同的 Image

正確做法是建立單一 Image，透過環境變數或 Config 在 Runtime 區分環境。

#### 4. 在 Dockerfile 中硬編碼設定

應使用 Entrypoint Script 在容器啟動時動態產生設定檔，這是官方 Image（MySQL、PostgreSQL）採用的模式。

### 基礎設施決策

#### VM 或裸機 (Bare Metal)？

- **兩者皆可**，選擇你熟悉的
- 建議先用 VM，後續再做裸機效能測試
- 隨著 Container 密度增加，需關注 Kernel 排程與網路設定

#### OS 選擇

Linux 發行版不如 Kernel 版本重要：

- **最低要求**：Kernel 3.10（Docker 最低支援）
- **建議版本**：Kernel 4.x 以上
- **無特別偏好時**：選擇 Ubuntu（預設 4.x Kernel、長期支援、文件豐富）

> Container 相關創新持續影響 Linux Kernel 發展，舊版 Kernel 可能缺少重要的容器功能與修正。

#### Base Image 選擇

- 從你現有 VM 使用的發行版開始
- 不要為了 Image 大小而強迫改用 Alpine
- 團隊標準化：可建立 Intermediate Image 作為團隊基礎

### Swarm 架構設計

#### Baby Swarm（單節點）

```bash
docker swarm init
```

單節點 Swarm 仍有價值：
- Secrets 管理
- Configs
- Service 自動重啟
- Health Check 支援

#### 三節點 Swarm

**最低的高可用性配置**，所有節點同時擔任 Manager 與 Worker。

#### 五節點 Swarm（Big Swarm）

適合小型團隊的高可用方案：
- 可容忍兩個 Manager 故障
- 可進行一次維護仍保持高可用

#### 大規模 Swarm

- Manager 與 Worker 分離
- Manager 放置於安全網段（VLAN/Security Group）
- Worker 可有不同硬體規格、網段、可用區
- 使用 Constraints 控制 Service 部署位置

```
┌─────────────────────────────────┐
│         Secure Enclave          │
│  ┌───┐  ┌───┐  ┌───┐           │
│  │ M │  │ M │  │ M │           │
│  └───┘  └───┘  └───┘           │
└─────────────────────────────────┘
              │
    ┌─────────┼─────────┐
    ▼         ▼         ▼
 ┌───┐     ┌───┐     ┌───┐
 │ W │     │ W │     │ W │
 └───┘     └───┘     └───┘
```

### 寵物 vs 家畜 (Pets vs Cattle)

**不要把你的節點變成寵物**：

- 不要在主機上 Clone Git Repository
- 不要在主機上安裝特殊工具
- 節點應保持：安裝 Docker → 加入 Swarm → 部署 Container

> 所有問題排解與測試都應在 Container 中進行，而非主機上。

### 何時需要多個 Swarm？

**不需要多個 Swarm的情況**：
- 節點數量限制（已測試上萬節點）
- Service 數量限制
- 團隊規模限制

**需要多個 Swarm 的情況**：
- Ops 團隊需要練習環境（在 Production 前犯錯）
- 管理邊界需求（不同辦公室各自管理）
- RBAC 需求（Docker API 預設是全有或全無）

### 外包基礎設施元件

避免「Not Invented Here」症候群，以下元件可考慮使用託管服務：

- **Registry** — Docker Hub、AWS ECR、Quay
- **Log 集中化** — ELK、Splunk
- **Monitoring** — Datadog、Prometheus

> Open Source 雖然免費，但通常是用便利性換取的，沒有真正的免費。

### 單 Container 單 VM 也是合理選擇

若專案時程緊迫，無法完成編排系統：

> 1 VM + 1 Container 仍然是合法的架構決策，這比完全沒用 Container 更好。

這也是 Hyper-V Containers 和 Linux runV 的運作模式。

## 💡 重點摘要

- 限制第一個容器專案的範疇，專注於 Dockerfile 品質
- Image 大小不是首要問題，版本固定與文件完整性更重要
- Swarm 從小開始，隨專案成長而擴展
- 節點應保持乾淨，所有操作都在 Container 內進行
- 單 Container 單 VM 是合理的過渡架構

## 🔑 關鍵字

Dockerfile, Swarm, Anti-pattern, Volume, Kernel
