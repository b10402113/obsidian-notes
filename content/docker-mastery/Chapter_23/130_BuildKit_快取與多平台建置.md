# BuildKit 快取與多平台建置

## 📝 課程概述

本單元介紹如何在 GitHub Actions 中啟用 BuildKit 快取，以及如何建置多平台 Image（如同時支援 amd64 與 arm64）。這兩個功能可以大幅提升建置效率並擴展 Image 的適用範圍。

---

## 核心觀念與實作解析

### BuildKit 簡介

**BuildKit** 是新一代的 Docker 建置引擎，已經存在 4-5 年：

- 建置出的 Image 仍符合標準 OCI 格式
- 建置速度更快
- 功能更豐富（如平行建置、更好的快取機制）

---

### 啟用 BuildKit 快取

#### 問題：CI 環境的快取挑戰

在本地環境，Docker 會自動快取 Layer，下次建置時直接複用。但在 CI 環境：

- 每次 Workflow 執行都會在**新的 VM** 上進行
- VM 在 Workflow 結束後會被銷毀
- 無法保留快取

#### 解決方案：GitHub Cache API

從 2021 年起，BuildKit 可以透過 **GitHub Cache API** 儲存快取：

```yaml
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v2

- name: Build and push
  uses: docker/build-push-action@v4
  with:
    context: .
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

#### 運作機制

```
建置 → 上傳快取到 GitHub Cache API
              ↓
下次建置 ← 從 GitHub Cache API 拉取快取
```

> **效益**：大幅減少重複建置相同 Layer 的時間，特別是 `npm install` 或 `pip install` 這類耗時步驟。

---

### 多平台建置（Multi-platform）

#### 為什麼需要多平台 Image？

隨著 Apple M1/M2（ARM 架構）、Raspberry Pi、雲端 ARM 實例的普及，越來越多場景需要 **arm64** 版本的 Image：

- 開發者使用 Apple Silicon（M1/M2）
- 生產環境使用 AWS Graviton（ARM）
- IoT 裝置使用 ARM 架構

#### 實作方式

```yaml
- name: Set up QEMU
  uses: docker/setup-qemu-action@v2

- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v2

- name: Build and push
  uses: docker/build-push-action@v4
  with:
    context: .
    platforms: linux/amd64,linux/arm64
    push: true
    tags: user/image:latest
```

#### QEMU 是什麼？

- **QEMU** 是一個模擬器，可以模擬不同的 CPU 架構
- 在 Docker Desktop 中已經內建
- 在 GitHub Actions 中需要手動啟用

> **建置過程**：BuildKit 會**平行建置**多個平台的 Image，最後打包成一個 Manifest List（也稱為 Fat Manifest），讓 `docker pull` 能自動選擇適合當前架構的 Image。

---

### 平台清單格式

`platforms` 參數使用標準的 OCI 平台格式，與 Docker Hub 上顯示的一致：

```
linux/amd64
linux/arm64
linux/arm/v7
```

---

## 💡 重點摘要

- **BuildKit 是新一代 Docker 建置引擎，速度更快、功能更豐富。**
- **GitHub Actions 可透過 GitHub Cache API 儲存 BuildKit 快取，解決 CI 環境無法保留快取的問題。**
- **多平台建置透過 QEMU 模擬不同 CPU 架構，BuildKit 會平行建置所有平台。**
- **多平台 Image 使用 Manifest List 打包，docker pull 時自動選擇適合的架構。**

---

## 🔑 關鍵字

BuildKit, Cache API, QEMU, Multi-platform, arm64
