# Docker 網路 CLI 管理操作

## 📝 課程概述

本單元將深入探討 Docker 網路的 CLI 管理命令。我們會學習如何建立、查看、連接虛擬網路，並理解自訂網路與預設 bridge 網路的關鍵差異，特別是 DNS 解析功能的應用。

## 核心觀念與實作解析

### Docker 網路管理命令總覽

| 命令 | 用途 |
|------|------|
| `docker network ls` | 列出所有網路 |
| `docker network inspect` | 查看網路詳細資訊 |
| `docker network create` | 建立新網路 |
| `docker network connect` | 將 Container 連接到網路 |
| `docker network disconnect` | 將 Container 從網路斷開 |

---

### 查看現有網路

```bash
docker network ls
```

**預設的三個網路**：

| 網路名稱 | 說明 |
|---------|------|
| **bridge** | 預設網路，透過 NAT 連接外部網路 |
| **host** | 直接使用 Host 網路，跳過虛擬網路層 |
| **none** | 無網路連接 |

> 在不同 OS 上，bridge 可能顯示為 `docker0` 或 `bridge`，兩者意義相同。

---

### 查看網路詳細資訊

```bash
docker network inspect bridge
```

輸出包含：
- 子網段（Subnet）：預設為 `172.17.x.x`
- 閘道（Gateway）
- 已連接的 Container 及其 IP 位址

---

### 建立自訂網路

```bash
docker network create my_app_net
```

**指定 Driver**：

```bash
docker network create --driver bridge my_app_net
```

> `bridge` 是預設 Driver，適用於單機虛擬網路。進階功能如跨主機網路需要其他 Driver（如 Overlay）。

---

### 將 Container 連接到特定網路

**方式一：建立時指定**：

```bash
docker container run -d --name new_nginx --network my_app_net nginx
```

**方式二：對運行中的 Container 動態連接**：

```bash
docker network connect my_app_net webhost
```

> 這相當於在實體機器上「熱插拔」一張網卡。

**斷開連接**：

```bash
docker network disconnect my_app_net webhost
```

---

### Container 可以連接多個網路

一個 Container 可以同時連接多個虛擬網路，類似實體機器有多張網卡：

```bash
# webhost 原本在 bridge 網路
# 執行 connect 後，它會同時擁有 bridge 和 my_app_net 兩個網路的 IP
docker network connect my_app_net webhost

# 查看結果
docker container inspect webhost
```

---

### 自訂網路的關鍵優勢：內建 DNS 解析
![[Pasted image 20260414194344.png|700]]

**這是自訂網路與預設 bridge 網路的最大差異**。
![[Pasted image 20260414194436.png|700]]

在自訂網路中，Container 可以**直接使用名稱互相通訊**：

```bash
# 在 my_app_net 網路中建立兩個 Container
docker container run -d --name my_nginx --network my_app_net nginx
docker container run -d --name new_nginx --network my_app_net nginx

# 測試 DNS 解析
docker container exec -it my_nginx ping new_nginx
# 成功！可以透過名稱 ping 到另一個 Container
```

**預設 bridge 網路的限制**：
- 沒有內建 DNS 解析
- Container 只能用 IP 互相通訊
- 可用 `--link` 手動建立連結（但不建議）

> **最佳實踐**：為每個應用建立專屬的自訂網路，而非使用預設 bridge。
> Compose 會自動新建網路
![[Pasted image 20260414194752.png|700]]
---

### 安全優勢

使用虛擬網路隔離不同應用：
![[Pasted image 20260414194138.png|700]] 

- 只有使用 `-p` 的 Container 會暴露到外部
- 同一虛擬網路內的 Container 可自由通訊
- 不同虛擬網路的 Container 完全隔離

> 這比傳統 VM 環境更容易管理網路安全邊界。

---
![[Pasted image 20260414194204.png|700]]
## 💡 重點摘要

- **自訂網路提供內建 DNS，Container 可用名稱互相通訊；預設 bridge 沒有此功能。**
- **`docker network connect/disconnect` 可動態調整運行中 Container 的網路連接。**
- **一個 Container 可連接多個網路，取得多個 IP 位址。**
- **建議為每個應用建立專屬網路，達到更好的隔離與管理。**

## 🔑 關鍵字

Network, Bridge, DNS, Driver, Alias
