# Container 生命週期與持久化資料

## 📝 課程概述

本單元是 Persistent Data 章節的開篇，我們將探討「為什麼持久化資料在 Container 環境中會成為問題」。透過理解 **Immutable Infrastructure** 與 **Ephemeral Container** 的設計哲學，我們會明白 Docker 如何透過 Data Volumes 與 Bind Mounts 來解決資料保存的挑戰。

---

## 核心觀念與實作解析

### Container 的設計哲學：Immutable 與 Ephemeral

在進入 Persistent Data 之前，我們必須先理解 Container 的核心設計理念：

- **Immutable（不可變）**：一旦 Container 開始運行，我們不應該去改變它內部的東西
- **Ephemeral（短暫的、可拋棄的）**：Container 被設計成可以隨時被丟棄，再從 Image 重新建立一個新的

> 這不是 Container 的技術限制，而是**設計目標與最佳實踐**。

這就是所謂的 **Immutable Infrastructure** 概念：當需要設定變更或版本升級時，我們不直接修改運行中的 Container，而是重新部署一個全新的 Container。

這麼做的好處是：
- 提高系統的可靠性與一致性
- 讓變更變得可重現（Reproducible）

---

### 那問題來了：應用程式產生的獨特資料怎麼辦？

這就是**取捨（Trade-off）**所在。當你的應用程式運行時，它會產生獨特的資料：

- Database 資料
- Key-Value Store 內容
- 任何寫入檔案的資料

**理想上，Container 不應該把「應用程式 binary」與「獨特資料」混在一起。** 這就是 Separation of Concerns（關注點分離）的原則。

> 當我們用新版本的 Image 重新建立 Container 時，希望我們的獨特資料仍然在原地，完好無損。

---

### Container 的檔案系統生命週期

這裡有個重要的觀念澄清：

**Container 內的檔案變更預設是持續存在的**，直到我們「移除」Container。

- 停止 Container（`docker container stop`）：檔案變更不會消失
- 重啟 Host 主機：檔案變更不會消失
- **移除 Container（`docker container rm`）**：Container 的 UFS（Union File System）層消失，檔案變更才會消失

> 問題在於：我們希望能夠隨心所欲地移除並重建 Container，但同時又需要保存資料。

---

### 什麼是 Persistent Data？

這個問題在業界被稱為 **Persistent Data（持久化資料）**。在 Container 出現之前，我們其實不太需要這個名詞，因為傳統 Server 本來就是持久運行的——想想那些跑了好幾年的老 Server。

但在 Container 與 Auto-scaling 的世界裡，Persistent Data 變成了獨特的挑戰。Docker 提供了兩種解決方案：

| 方案           | 說明                                   |
| ------------ | ------------------------------------ |
| Data Volumes | 在 Container 的 UFS 之外建立特殊位置存放資料       |
| Bind Mounts  | 將 Host 的目錄或檔案映射（Mount）進 Container 內部 |

無論哪種方式，Container 內部看起來都只是一個普通的本地檔案路徑。

---

### Data Volumes 的運作機制

#### 在 Dockerfile 中定義 Volume

讓我們以官方 MySQL Image 為例。老師習慣直接去 Docker Hub 查看官方 Repository 的 Dockerfile，因為它們通常包含最佳實踐。

```dockerfile
VOLUME /var/lib/mysql
```

這行指令告訴 Docker：當從這個 Image 啟動新 Container 時，要建立一個新的 Volume 位置，並掛載到 Container 內的 `/var/lib/mysql`。

**關鍵點：Volume 需要手動刪除**

> 你無法透過移除 Container 來清除 Volume。這是一種「保險」設計——因為 Volume 的整個目的就是標示「這些資料特別重要」。

---

#### 實際操作：查看 Volume 資訊

**步驟 1：Pull Image 並查看 Volume 設定**

```bash
docker pull mysql
docker image inspect mysql
```

在輸出的 `Config` 區塊中可以看到 `Volumes` 設定。

**步驟 2：啟動 Container**

```bash
docker container run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql
```

> MySQL Image 需要環境變數才能啟動，這裡使用 `MYSQL_ALLOW_EMPTY_PASSWORD` 範例。

**步驟 3：檢查 Container 的掛載資訊**

```bash
docker container inspect mysql
```

在 `Mounts` 區塊可以看到：
- Container 獲得一個獨特的 Host 路徑
- 該路徑被掛載到 Container 內的 `/var/lib/mysql`

**步驟 4：查看 Volume 列表**

```bash
docker volume ls
```

你會看到一個帶有隨機 ID 的 Volume。

**步驟 5：檢查 Volume 詳情**

```bash
docker volume inspect <volume_id>
```

---

### Named Volumes：讓 Volume 更容易管理

隨機 ID 的問題在於——當你有多個 Volume 時，根本無法分辨哪個是哪個。這就是 **Named Volumes** 的用武之地。

#### 在執行時指定 Named Volume

```bash
docker container run -d --name mysql -v mysql-data:/var/lib/mysql mysql
```

`-v` 參數格式：`<volume_name>:<container_path>`

- 冒號左邊是 Volume 名稱
- 冒號右邊是 Container 內的路徑

> Named Volume 的好處：名稱有意義、容易辨識、路徑也更友善。

---

### Container 移除後 Volume 仍然存在

讓我們驗證 Volume 的持久性：

```bash
# 停止並移除 Container
docker container stop mysql mysql2
docker container rm mysql mysql2

# Volume 仍然存在
docker volume ls
```

**這證明了資料已與 Container 生命週期解耦。**

---

### 為什麼需要 `docker volume create`？

既然可以在 Dockerfile 或 `docker run` 時建立 Volume，為什麼還需要手動建立？

```bash
docker volume create --help
```

唯一需要在建立時指定的情況是：
- 使用不同的 Driver（Driver Plugins）
- 指定 Driver Options（`-o`）
- 加上 Labels（在 Production 章節會討論）

> 大多數本地開發場景，直接在 Dockerfile 或 `docker run` 指定就足夠了。

---

## 💡 重點摘要

- **Container 的設計哲學是 Immutable + Ephemeral，這是最佳實踐而非技術限制。**
- **Persistent Data 的解法是將資料存放在 Container UFS 之外，透過 Volumes 或 Bind Mounts 實現。**
- **Volume 不會隨 Container 移除而消失，需要手動刪除——這是刻意設計的「保險機制」。**
- **Named Volumes 解決了隨機 ID 難以辨識的問題，建議為專案命名以便管理。**
- **大多數情況下不需要手動 `docker volume create`，除非需要指定特殊 Driver 或 Options。**

---

## 🔑 關鍵字

Volume, Named Volume, Bind Mount, UFS, Immutable Infrastructure
