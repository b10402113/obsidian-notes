# Image Layer 與 Union File System

## 📝 課程概述

本單元深入探討 Docker Image 的核心技術：**Layer（層）** 與 **Union File System**。理解 Layer 的運作機制，能幫助我們最佳化 Image 大小、善用 Build Cache，並理解 Container 與 Image 之間的關係。

---

## 核心觀念與實作解析

### 什麼是 Image Layer？

Docker Image 不是一個巨大的單一檔案，而是由多個 **Layer（層）** 堆疊而成。

#### Union File System 的概念

Union File System 將一系列檔案系統變更，呈現為一個完整的檔案系統。

```
┌─────────────────────────┐
│       Layer 4           │  ← COPY source code
│     (新增/修改檔案)       │
├─────────────────────────┤
│       Layer 3           │  ← EXPOSE 80
│     (Metadata 變更)      │
├─────────────────────────┤
│       Layer 2           │  ← RUN apt install. 
│     (新增 123MB 檔案)     │
├─────────────────────────┤
│       Layer 1           │  ← FROM debian
│     (Base Image)         │
├─────────────────────────┤
│       scratch           │  ← 空白起點
└─────────────────────────┘
```

> **每個 Image 都從 `scratch`（空白層）開始，後續的每個變更都會新增一個 Layer。**
---

### Layer 的唯一識別：SHA Hash

每個 Layer 都有獨特的 **SHA Hash**，這帶來兩個重要好處：

1. **唯一性保證** — 相同內容的 Layer 必有相同的 SHA
2. **去重儲存** — 相同 Layer 只儲存一份

#### 範例：兩個 Image 共用 Layer
![[Pasted image 20260418092745.png|700]]

```
Image A (Apache)          Image B (MySQL)
    │                         │
    ▼                         ▼
┌─────────┐              ┌─────────┐
│ Layer 3 │              │ Layer 3 │
│ COPY src│              │ COPY cnf│
├─────────┤              ├─────────┤
│ Layer 2 │              │ Layer 2 │
│ apt ins │              │ apt ins │
├─────────┤              ├─────────┤
│         │    共用層     │         │
│ debian:jessie ◄────────►debian:jessie
│         │   (SHA 相同)  │         │
└─────────┘              └─────────┘
```

**儲存空間節省：** 系統只會儲存一份 `debian:jessie` Layer，兩個 Image 共用。

---

### `docker history` 指令

使用 `docker history` 可以查看 Image 的 Layer 組成：

```bash
docker history nginx:latest
```

輸出會顯示：
- 每個 Layer 的大小
- 建立時間
- 造成的變更（CREATE BY）

> 注意：`MISSING` 不是錯誤，只是表示該 Layer 不是獨立的 Image，沒有自己的 Image ID。

---

### `docker image inspect` 指令

查看 Image 的 Metadata：

```bash
docker image inspect nginx
```

**回傳的重要資訊：**

| 欄位 | 說明 |
|------|------|
| `ExposedPorts` | Image 預期開放的 Port |
| `Env` | 環境變數 |
| `Cmd` | 預設執行指令 |
| `Architecture` | 架構（如 amd64） |
| `Os` | 目標作業系統（如 linux） |

---

### Container 與 Layer 的關係

#### Container 是 Image 之上的讀寫層

當你從 Image 啟動 Container 時，Docker 會在 Image 之上新增一個 **Read/Write Layer**：

```
┌─────────────────────────────┐
│   Container Layer (R/W)     │  ← Container 的變更都在這
├─────────────────────────────┤
│    Image Layer 3 (R/O)      │
├─────────────────────────────┤
│    Image Layer 2 (R/O)      │
├─────────────────────────────┤
│    Image Layer 1 (R/O)      │
└─────────────────────────────┘
```

**關鍵點：**
- Image 的所有 Layer 都是 **Read-Only（唯讀）**
- Container 只有自己的 Read/Write Layer
- 多個 Container 可共用同一個 Image，各自有獨立的 Read/Write Layer

---

### Copy-on-Write 機制

當 Container 修改 Image 中的檔案時，會觸發 **Copy-on-Write**：

```
原始狀態：
┌─────────────────────┐
│ Container (empty)   │
├─────────────────────┤
│ Image: config.conf  │  ← 檔案存在 Image Layer
└─────────────────────┘

修改檔案後：
┌─────────────────────┐
│ Container           │
│ config.conf (修改版) │  ← 檔案被複製到此
├─────────────────────┤
│ Image: config.conf  │  ← 原始檔案不變
└─────────────────────┘
```

**Copy-on-Write 的運作：**
1. 當 Container 要修改 Image 中的檔案
2. 系統將該檔案複製到 Container 的 Read/Write Layer
3. Container 修改的是複本，原始檔案不變

> 這意味著 Container 的 Layer 只包含**與 Image 不同的檔案**，節省儲存空間。

---

## 💡 重點摘要

- **Docker Image 由多個 Read-Only Layer 堆疊而成，每個 Layer 有獨特的 SHA Hash。**
- **Union File System 將多層變更呈現為單一檔案系統，Layer 間可共用，節省儲存空間與傳輸時間。**
- **Container 啟動時，Docker 在 Image 之上新增 Read/Write Layer，Container 的變更都在這層。**
- **Copy-on-Write 機制：修改 Image 檔案時，系統會將檔案複製到 Container Layer 再修改。**
- **`docker history` 查看 Layer 組成；`docker image inspect` 查看 Metadata。**

---

## 🔑 關鍵字

Layer, Union File System, SHA, Copy-on-Write, Build Cache, Metadata
