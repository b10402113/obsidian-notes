# Dockerfile 指令類型回顧

## 📝 課程概述

本單元是 ENTRYPOINT 章節的前導課程，老師帶領我們複習已學過的七個 Dockerfile 指令，並引入兩個重要的分類維度：**Buildtime vs Runtime** 與 **Overwrite vs Additive**。理解這些分類是掌握 ENTRYPOINT 與 CMD 關係的基礎。

---

## 核心觀念與實作解析

### 已學過的七個 Dockerfile 指令

| 指令 | 用途 |
|------|------|
| `FROM` | 指定 Base Image（必須） |
| `ENV` | 設定環境變數 |
| `WORKDIR` | 切換/建立工作目錄 |
| `COPY` | 複製本地檔案到 Image |
| `RUN` | Build 時執行指令 |
| `EXPOSE` | 宣告監聽 Port |
| `CMD` | Container 啟動時的預設指令 |

---

### 維度一：Buildtime vs Runtime

每個 Dockerfile 指令在「何時生效」這件事上有根本差異。

#### Buildtime 指令

在 `docker build` 時執行，結果儲存在 Image Layer 中：

- `RUN` — 執行指令，變更檔案系統
- `COPY` — 複製檔案到 Image

> 這些指令的結果會永久儲存在 Image 中，Container 啟動時不會重新執行。

#### Runtime 指令

儲存在 Image Metadata 中，在 `docker run` 時才生效：

- `CMD` — 定義啟動指令
- `EXPOSE` — 宣告 Port（只是 Metadata）

#### 同時影響 Buildtime 與 Runtime

- `ENV` — 在 Build 時供後續 `RUN` 使用，Container 啟動時也會設定

**常見誤解：**

> 很多初學者以為 `CMD` 會在 Build 時執行，其實不會。`CMD` 只是儲存在 Metadata 中，Container 啟動時才執行。

---

### 維度二：Overwrite vs Additive

指令之間的「疊加」或「覆蓋」行為也不同。

#### Overwrite（覆蓋型）

只有最後一個指令生效，之前的會被覆蓋：

- `CMD` — 只有最後一個生效
- `WORKDIR` — 只有最後一個設定路徑生效
- `ENTRYPOINT` — 只有最後一個生效

#### Additive（疊加型）

多次使用會累加：

- `EXPOSE` — 所有宣告的 Port 都會保留
- `ENV` — 所有環境變數都會保留

**Pro Tip：**

> `EXPOSE` 可以在一行宣告多個 Port：`EXPOSE 80 443`。`ENV` 也可以一次設定多個變數。

---

### 指令分類速查表

老師提供了完整的 Cheat Sheet，整理如下：

| 指令 | 時機 | 行為 |
|------|------|------|
| `FROM` | Buildtime | Overwrite |
| `RUN` | Buildtime | Additive（每個建立 Layer） |
| `COPY` | Buildtime | Additive |
| `ADD` | Buildtime | Additive |
| `ENV` | Both | Additive |
| `WORKDIR` | Both | Overwrite |
| `ARG` | Buildtime | Overwrite |
| `EXPOSE` | Runtime | Additive |
| `CMD` | Runtime | Overwrite |
| `ENTRYPOINT` | Runtime | Overwrite |
| `VOLUME` | Runtime | Additive |
| `USER` | Runtime | Overwrite |
| `HEALTHCHECK` | Runtime | Overwrite |
| `ONBUILD` | Both | Additive |

> 不需要背誦這些分類，但要理解這些效應存在。當 Dockerfile 或 Container 行為不如預期時，可能是這些效應造成的。

---

### 本章節重點

本章節將深入探討 `ENTRYPOINT` 指令及其與 `CMD` 的關係。理解上述兩個維度是後續課程的基礎。

---

## 💡 重點摘要

- **Dockerfile 指令分為 Buildtime（Build 時執行）與 Runtime（Container 啟動時執行）。**
- **`CMD` 是 Runtime 指令，只在 Container 啟動時執行，Build 時不會執行。**
- **指令行為分為 Overwrite（覆蓋前值）與 Additive（累加）；`CMD` 是 Overwrite，`EXPOSE` 是 Additive。**
- **`ENV` 同時影響 Buildtime 與 Runtime，是特殊案例。**
- **理解這些分類有助於診斷 Dockerfile 的非預期行為。**

---

## 🔑 關鍵字

CMD, ENTRYPOINT, Buildtime, Runtime, Overwrite, Additive, Metadata
