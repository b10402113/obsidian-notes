# DNS Round Robin 測試

## 📝 課程概述

本單元將實作 Docker 的 DNS Round Robin 功能。我們會學習如何讓多個 Container 使用相同的 DNS 別名，實現簡易的負載平衡效果。這是在不使用完整負載平衡器的前提下，實現服務高可用性的基礎技巧。

## 核心觀念與實作解析

### 什麼是 DNS Round Robin？

DNS Round Robin 是一種簡易的負載平衡技術：

- 同一個 DNS 名稱對應多個 IP 位址
- DNS 查詢會輪流返回不同的 IP
- 客戶端每次查詢可能得到不同的結果

**現實案例**：`google.com` 背後有多部伺服器，DNS 會輪流返回不同的 IP。

---

### Docker 中的 DNS Round Robin

從 Docker Engine 1.11 開始，我們可以在自訂網路中使用 `--net-alias` 讓多個 Container 共用同一個 DNS 別名。

---

### 實作步驟

**步驟一：建立自訂網路**

```bash
docker network create dude
```

**步驟二：建立兩個 Elasticsearch Container**

```bash
docker container run -d --network dude --net-alias search elasticsearch:2
docker container run -d --network dude --net-alias search elasticsearch:2
```

關鍵選項：
- `--network dude`：連接到自訂網路
- `--net-alias search`：設定 DNS 別名為 `search`

> 兩個 Container 都使用相同的別名 `search`。

**步驟三：驗證 DNS 解析**

```bash
docker container run --rm --network dude alpine nslookup search
```

輸出會顯示兩個 IP 位址，代表 `search` 別名對應到兩個 Container。

**步驟四：測試 Round Robin**

```bash
docker container run --rm --network dude centos curl -s search:9200
```

多次執行這個命令，你會發現：

```json
// 第一次可能返回
{"name": "Interloper", ...}

// 第二次可能返回
{"name": "Mr. M", ...}

// 再執行可能又回到
{"name": "Interloper", ...}
```

> Elasticsearch 會隨機命名自己，這讓我們可以清楚辨別連到哪個 Container。

---

### 重要觀察

**Round Robin 不是精確輪詢**：

- 不會嚴格地「第一個、第二個、第一個、第二個」
- 可能連續兩次連到同一個，然後才切換
- 這是 DNS Round Robin 的特性，受 DNS 快取等因素影響

> DNS Round Robin 被稱為「窮人的負載平衡器」——簡單但不完美。

---

### 實際應用場景

| 場景 | 說明 |
|------|------|
| 開發/測試環境 | 同一伺服器上運行 dev 和 test 環境，使用相同別名 |
| 簡易 HA | 多個實例共用別名，提供基本的高可用性 |
| 服務發現 | 客戶端只需知道服務名稱，不需知道具體 Container |

---

### 為什麼不用 Container 名稱？

Container 名稱必須唯一，不能有多個 Container 同名。`--net-alias` 讓我們突破這個限制，多個 Container 可以共用別名。

---

## 💡 重點摘要

- **`--net-alias` 讓多個 Container 共用同一個 DNS 別名。**
- **DNS Round Robin 實現簡易負載平衡，但不是精確輪詢。**
- **必須在自訂網路中使用，預設 bridge 網路不支援此功能。**
- **Elasticsearch 的隨機命名特性方便驗證 Round Robin 效果。**

## 🔑 關鍵字

DNS, Round Robin, net-alias, Elasticsearch, Load Balancer
