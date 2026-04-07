# ENTRYPOINT 基本概念

## 📝 課程概述

本單元正式介紹 `ENTRYPOINT` 指令。雖然 `ENTRYPOINT` 和 `CMD` 都定義 Container 啟動時要執行的指令，但它們的行為有關鍵差異。老師透過實際操作 BusyBox Image，展示兩者在「覆蓋」行為上的不同。

---

## 核心觀念與實作解析

### ENTRYPOINT 與 CMD 的相似處

兩者都是 Runtime 指令，都定義 Container 啟動時執行的指令：

- Container 啟動時執行
- 只有一個生效（Overwrite）
- 必須至少有其中一個

> Docker 需要知道 Container 啟動時執行什麼，所以 `CMD` 或 `ENTRYPOINT` 至少要有一個。

---

### BusyBox Image 實作

#### 查看預設設定

```bash
docker inspect busybox
```

在 `Config` 區塊可以看到：
- `Cmd`: `["sh"]`
- `Entrypoint`: `null`

這表示 BusyBox 預設啟動一個 Shell。

#### 互動式執行

```bash
docker run -it busybox
```

進入 Container 後可以執行各種 Linux 工具，如 `hostname`、`date` 等。

---

### CMD vs ENTRYPOINT 的覆蓋行為

#### 使用 CMD

```dockerfile
FROM busybox:latest
CMD ["hostname"]
```

```bash
docker build -t hostname .
docker run hostname
# 輸出 Container ID

docker run hostname date
# 覆蓋 CMD，執行 date 指令
```

**CMD 的特性：** 可以輕易在 `docker run` 時覆蓋。

#### 使用 ENTRYPOINT

```dockerfile
FROM busybox:latest
ENTRYPOINT ["hostname"]
```

```bash
docker build -t entryhostname .
docker run entryhostname
# 輸出 Container ID

docker run entryhostname date
# 錯誤！date 參數被傳給 hostname，而非覆蓋指令
```

**ENTRYPOINT 的特性：** 無法用同樣方式覆蓋。

---

### 如何覆蓋 ENTRYPOINT？

需要使用 `--entrypoint` 選項：

```bash
docker run --entrypoint date entryhostname
```

> 覆蓋 `ENTRYPOINT` 需要更多打字，這是刻意的設計。

---

### 什麼時候用哪個？

#### 只用 CMD 的場景

> **如果只需要啟動單一程序，且選項很少變動，使用 `CMD` 即可。**

適合：
- Web Server（如 nginx）
- Database（如 mysql）
- 長時間執行的背景程序

這是 Docker Hub 官方 Image 的標準做法。

#### ENTRYPOINT 單獨使用的限制

`ENTRYPOINT` 單獨使用時，相較於 `CMD` **沒有任何優勢**。

它的真正威力在於 **與 CMD 組合使用**。

---

## 💡 重點摘要

- **`ENTRYPOINT` 和 `CMD` 都定義 Container 啟動指令，但覆蓋方式不同。**
- **`CMD` 可直接在 `docker run` 後加上指令覆蓋；`ENTRYPOINT` 需用 `--entrypoint` 選項。**
- **長時間執行的程序（Web Server、Database）使用 `CMD` 即可，這是官方 Image 的標準做法。**
- **`ENTRYPOINT` 單獨使用無優勢，真正威力在於與 `CMD` 組合。**
- **覆蓋 `ENTRYPOINT` 需要更多打字，是刻意的設計決策。**

---

## 🔑 關鍵字

ENTRYPOINT, CMD, Overwrite, BusyBox, Runtime, docker run
