# BuildKit 與 buildx:新世代映像檔建置工具

## 📝 課程概述

本單元介紹 Docker 19.03 版本的重要新功能 buildx,這是基於 BuildKit 的增強型建置工具。我們將學習如何使用 buildx 進行跨平台映像檔建置、理解 Manifest 的運作機制,以及掌握多架構映像檔的自動化建置流程。

## 核心觀念與實作解析

### BuildKit 與 buildx 的關係

#### 技術演進脈絡

**BuildKit** 是 Moby Project 的新世代建置引擎:
- 已存在約兩年,技術成熟
- 提供更快的建置速度
- 支援進階功能如平行建置、快取機制

**buildx** 是 Docker CLI 插件:
- 作為 BuildKit 的命令列介面
- 取代舊版的環境變數啟用方式
- 提供更豐富的功能集

**關鍵變革**:從設定環境變數 `DOCKER_BUILDKIT=1`,改為使用獨立的 `docker buildx` 命令。

### buildx 的核心優勢

#### 1. 多平台映像檔建置

**傳統做法的痛點**:
- 需手動建立 Manifest 檔案
- 逐一建置不同架構的映像檔
- 手動推送到 Registry
- 流程繁瑣且易出錯

**buildx 的解決方案**:
```bash
docker buildx build --platform linux/amd64,linux/arm64,linux/arm/v7 \
  -t username/app:latest --push .
```

單一指令即可:
- 同時建置 AMD64、ARM64、ARM v7 三種架構
- 自動建立 Manifest 檔案
- 推送至 Docker Hub

#### 2. 平行建置能力

**多節點建置**:
- 可在多台機器上同時執行建置
- 分散運算負載
- 加速大型專案建置

**本地多架構建置**:
- 在單一機器上使用 QEMU 模擬器
- 建置不同 CPU 架構的映像檔

#### 3. 內建快取機制

**BuildKit 的快取特性**:
- 智慧型層級快取
- 僅重建變更的部分
- 大幅縮短建置時間

### Manifest 檔案機制詳解

#### 什麼是 Manifest?

**Manifest** 是映像檔的「物料清單」(Bill of Materials):
- 記錄映像檔支援的架構平台
- 對應到不同架構的具體映像檔
- 讓 Docker Client 自動選擇正確版本

#### 運作流程

**使用者的視角**:
```bash
docker run python:3.9
```

**Docker 的運作邏輯**:
1. 查詢 Registry 取得 Manifest
2. 讀取本機架構資訊
3. 選擇對應架構的映像檔
4. 下載並執行

**建置者的視角**(使用 buildx):
1. 建置多個架構的映像檔
2. 自動產生 Manifest
3. 推送所有映像檔與 Manifest 到 Registry

### 實作:使用 buildx 建置多架構映像檔

#### 環境準備

**安裝 buildx**(若使用 19.03 穩定版):

1. 從 GitHub 下載:
```
https://github.com/docker/buildx
```

2. 放置到 Docker CLI 插件目錄:
```bash
cp buildx ~/.docker/cli-plugins/docker-buildx
chmod +x ~/.docker/cli-plugins/docker-buildx
```

**驗證安裝**:
```bash
docker buildx version
```

#### 建立 Builder 實例

**預設 Builder 的限制**:
- 使用 Docker Driver
- 基於 Moby Storage
- **不支援多平台建置**

**建立新的 Builder**:
```bash
docker buildx create --name mybuilder --driver docker-container
docker buildx use mybuilder
```

**docker-container Driver 的特點**:
- 使用 BuildKit 完整功能
- 支援多平台建置
- 在容器中運行建置程序

#### 執行多平台建置

**基本語法**:
```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t username/app:v1 \
  --push .
```

**重要參數**:
- `--platform`:指定目標架構(可多選)
- `-t`:映像檔標籤
- `--push`:建置完成後直接推送
- `.`:建置上下文路徑

#### 驗證建置結果

**檢查 Manifest**:
```bash
docker buildx imagetools inspect username/app:v1
```

**輸出範例**:
```
Manifests:
  Name:      username/app:v1@sha256:...
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/amd64

  Name:      username/app:v1@sha256:...
  MediaType: application/vnd.docker.distribution.manifest.v2+json
  Platform:  linux/arm64
```

### Docker CLI 插件系統

#### 插件的運作機制

**插件目錄位置**:
```
~/.docker/cli-plugins/
```

**插件檔案規範**:
- 可執行檔(通常為 Shell Script 或 Go Binary)
- 檔名格式:`docker-<command-name>`

**必要支援**:
```bash
#!/bin/bash
if [ "$1" = "docker-cli-plugin-metadata" ]; then
  cat <<EOF
{
  "SchemaVersion": "0.1.0",
  "Vendor": "Your Name",
  "Version": "v1.0.0",
  "ShortDescription": "Your plugin description"
}
EOF
  exit 0
fi
# 實際功能實作
```

#### 啟用實驗性功能

**修改 Docker 配置**:
```json
{
  "experimental": "enabled"
}
```

**配置檔位置**:
- Linux/macOS: `~/.docker/config.json`
- Windows: `%USERPROFILE%\.docker\config.json`

### 實際應用場景

#### 1. IoT 與邊緣運算

為 Raspberry Pi、AWS A1 實例建置 ARM 映像檔:
```bash
docker buildx build --platform linux/arm64 \
  -t myapp:arm64 .
```

#### 2. 混合雲架構

支援多種硬體架構的統一映像檔標籤:
```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t registry.company.com/app:latest \
  --push .
```

#### 3. CI/CD 自動化

整合到 GitHub Actions、GitLab CI 等流程:
- 自動建置多架構映像檔
- 統一版本管理
- 簡化部署流程

### 穩定性評估

#### 功能成熟度

**BuildKit 本身**:
- 已發展兩年
- 廣泛使用於生產環境
- 穩定性高

**buildx 插件**:
- Docker 19.03 中仍標記為實驗性
- 在 Linux 平台完全支援
- macOS/Windows 部分功能仍在優化

#### 導入建議

**建議使用場景**:
- CI/CD 環境(不影響生產容器運行)
- 多架構映像檔建置需求
- 需要更快建置速度的專案

**注意事項**:
- 從開發環境開始測試
- 驗證建置結果正確性
- 觀察社群回饋與更新

## 💡 重點摘要

- buildx 是 BuildKit 的 CLI 插件,提供多平台建置、平行建置、快取優化等功能
- 使用 `docker-container` Driver 才能執行多架構建置,預設 Docker Driver 不支援
- Manifest 檔案記錄映像檔的架構支援資訊,讓 Docker 自動選擇正確版本
- buildx 可透過單一指令完成多架構建置、Manifest 產生與 Registry 推送

## 🔑 關鍵字

BuildKit, buildx, Manifest, Multi-platform, Docker CLI Plugin
