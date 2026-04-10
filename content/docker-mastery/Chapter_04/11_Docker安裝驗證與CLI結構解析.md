# Docker 安裝驗證與 CLI 結構解析

## 📝 課程概述

本單元是「Container 操作」章節的起點，我們將確認 Docker 環境是否正確安裝，並深入了解 Docker CLI 的命令結構演進。理解新舊命令格式的差異，將有助於我們在後續課程中更順暢地操作 Docker。

## 核心觀念與實作解析

### 確認 Docker 環境正確運行

在開始學習 Container 操作之前，我們必須確保 Docker 已正確安裝並能正常運作。這部分只需要兩個關鍵命令：
![[Pasted image 20260408191253.png|700]]
**1. `docker version` — 驗證 Client 與 Server 通訊**

這個命令會回傳兩個部分的版本資訊：
- **Client**：你正在使用的 CLI 工具
- **Server（也稱為 Engine）**：在背景執行的 Docker Daemon

> 在 Windows 上通常稱為 Service，在 Mac/Linux 上則稱為 Daemon。

當你執行 `docker version` 時，CLI 實際上是在與本機的 Server 通訊並取得其版本資訊。**如果能看到 Server 的版本資訊，就代表 CLI 可以正常與 Server 溝通**。

如果遇到錯誤，可能的原因包括：
- 在 Linux 上需要加上 `sudo`
- Docker 本身尚未啟動
![[Pasted image 20260408191431.png|700]]
**2. `docker info` — 取得詳細配置資訊**

這個命令會回傳更豐富的引擎配置資訊，包括：
- 運行中的 Container 數量
- 已儲存的 Image 數量
- 其他各種配置值

> 很多配置值目前可能看不懂，這很正常，後續課程會逐一解釋。

---

### Docker CLI 命令結構的演進

Docker 的命令結構在 2017 年初經歷了一次重要的重組。讓我們來理解為什麼以及如何演進：

**舊式命令格式**
```
docker <command> [options]
```
例如：`docker run`

**新式命令格式（Management Commands）**
```
docker <management-command> <subcommand> [options]
```
例如：`docker container run`

> **為什麼要改？** Docker 發現命令數量愈來愈多，原本的扁平式結構變得難以管理，因此引入了「管理命令」的分層架構。

![[Pasted image 20260408191605.png|700]]

---

### 新舊並存的設計哲學

Docker 非常重視**向後相容性（Backwards Compatibility）**：

| 項目 | 說明 |
|------|------|
| 舊命令 | 仍然可用，例如 `docker run` |
| 新命令 | 新功能將採用 `docker <command> <subcommand>` 格式 |
| 建議 | 兩種方式都學，因為舊命令會持續運作很長一段時間 |

**以 `docker run` 為例**：
- 舊格式：`docker run`
- 新格式：`docker container run`（兩者功能完全相同）

---

## 💡 重點摘要

- **`docker version` 是在新環境上第一個要執行的命令，用於確認 Client 與 Server 能正常通訊。**
- **`docker info` 提供更詳細的引擎配置資訊，適合用來檢查環境設定是否符合預期。**
- **Docker CLI 採用「管理命令」新架構，但舊命令因向後相容性仍可繼續使用。**
- **本課程會同時展示新舊兩種寫法，讓學員能理解並靈活運用。**

## 🔑 關鍵字

Docker, CLI, Daemon, Container, Image
