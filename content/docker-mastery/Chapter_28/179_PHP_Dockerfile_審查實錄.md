# PHP Dockerfile 審查實錄

## 📝 課程概述

本單元審查了 Charles 提交的 PHP Apache Dockerfile，這是一個已經在 Production 環境運行的真實案例。老師針對 Cache Busting、Image 大小優化、套件版本固定等議題提供詳細的改進建議，幫助學員理解如何在 Real World 場景中優化 Dockerfile。

---

## 核心觀念與實作解析

### 審查標的：PHP Apache 客製化 Image

這個 Dockerfile 基於自建的 PHP Apache Image，用於 Azure 環境。讓我們來看看其中的關鍵問題與改進建議。

### 問題一：`apt-get update` 與 Install 分離

#### 原始寫法

```dockerfile
RUN apt-get update || apt-get update  # 重試機制

RUN apt-get install -y package1 package2 ...
```

#### 問題分析

這樣寫會導致 **Cache Busting 問題**：

- `apt-get update` 會下載所有套件的 metadata cache
- 如果下面的 install line 被修改（新增套件、改版本），會使用**舊的 cache**
- 長期下來，你安裝的永遠是舊版本的套件

> **Docker Cache 機制**：當某一行被修改，該行及其後續所有行都會重新執行。但如果 update 和 install 分開，install line 觸發 rebuild 時，會使用舊的 update cache。

#### 正確寫法

```dockerfile
RUN apt-get update && apt-get install -y \
    package1 \
    package2 \
    && rm -rf /var/lib/apt/lists/*
```

把 update 和 install 放在同一個 `RUN` 指令中，確保每次安裝都使用最新的 metadata。

---

### 問題二：下載的 deb 檔案殘留

#### 原始寫法

```dockerfile
RUN wget http://example.com/package.deb
RUN dpkg -i package.deb
RUN rm package.deb
```

#### 問題分析

這是一個經典錯誤：**Docker Layer 是唯讀的**。

- 第一行建立一個 Layer，包含 `package.deb`
- 第三行雖然刪除檔案，但只是在新的 Layer 加上「刪除標記」
- **原始檔案仍然存在於唯讀 Layer 中**，Image 大小不會減少

#### 正確寫法

```dockerfile
RUN wget http://example.com/package.deb \
    && dpkg -i package.deb \
    && rm package.deb
```

所有操作在**同一個 RUN 指令**中完成，檔案在同一個 Layer 內被刪除，不會殘留在 Image 中。

> 可以改用 `ADD` 指令自動下載檔案，但同樣需要在同一行中處理清理動作。

---

### 問題三：沒有固定套件版本

#### 風險說明

```dockerfile
RUN apt-get install -y libapache2-mod-php
```

這樣寫會安裝「當下最新版本」，但有一天可能會：

1. 開發環境 Build 成功
2. Production 環境 Build 時安裝了新版本
3. 新版本有 bug，Production 直接爆炸

老師分享了自己的慘痛經驗：MongoDB PHP driver 更新了一個 minor version，導致 Production app 全面崩潰。

#### 正確寫法

```dockerfile
RUN apt-get install -y \
    package1=1.2.3 \
    package2=4.5.6
```

**強烈建議固定所有 Production 相關套件的版本**，並定期（如每月）更新。

---

### 問題四：多個 COPY 指令可以合併

#### 優化前

```dockerfile
COPY file1.crt /etc/ssl/certs/
COPY file2.key /etc/ssl/certs/
```

#### 優化後

```dockerfile
COPY file1.crt file2.key /etc/ssl/certs/
```

只要目標目錄相同，就可以用單一 COPY 指令複製多個檔案，減少 Layer 數量。

---

### 問題五：SSL 憑證不應寫入 Image

將 SSL 憑證直接 COPY 進 Image 不是最佳實踐：

- Image 應該保持「無敏感資訊」
- 憑證應該透過 Orchestrator（Swarm Config / Kubernetes Secret）在 Runtime 掛載

---

### 其他審查要點

#### ENTRYPOINT 與 CMD 的可讀性

如果 Base Image 已經定義了 `CMD`，繼承的 Image 可以不重複定義。但老師建議：

> 為了讓團隊成員容易理解，建議在 Dockerfile 底部明確寫出 `ENTRYPOINT` 和 `CMD`，即使它們只是重複 Base Image 的設定。

這對於不熟悉 Docker 的團隊成員特別有幫助，他們不需要去查 Base Image 的歷史設定。

---

## 💡 重點摘要

- **`apt-get update` 與 `apt-get install` 必須在同一個 RUN 指令中，避免使用過期的 package cache。**
- **在同一個 RUN 指令中清理暫存檔案，否則檔案仍會殘留在唯讀 Layer 中。**
- **固定所有 Production 套件的版本，避免因版本更新導致非預期的 Production 問題。**
- **敏感資訊（如 SSL 憑證）應透過 Orchestrator 在 Runtime 掛載，而非寫入 Image。**

---

## 🔑 關鍵字

Cache Busting, Layer, Version Pinning, ADD, COPY
