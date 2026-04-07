# Docker Rootless Mode 無 Root 運行模式

## 📝 課程概述

本單元介紹 Docker Rootless Mode,這是一項相對較新的功能,允許 Docker Daemon 以非 root 身份運行。我們將探討其適用場景、技術限制,以及在 Windows 容器環境中的安全實踐差異。

## 核心觀念與實作解析

### Docker Rootless Mode 核心概念

#### 什麼是 Rootless Mode?

**Rootless Mode** 讓 Docker Engine 以一般使用者身份運行,而非傳統的 root 身份:

- **Docker Daemon**:以非 root 身份運行
- **容器程序**:同樣以非 root 身份運行
- **安全提升**:完全消除 root 權限風險

**技術演進**:這是在近幾年內新增的功能,代表 Docker 安全性的重要進展。

#### 主要技術限制

Rootless Mode 並非適用所有場景,關鍵限制包括:

##### 1. 網路功能限制

**無法建立自訂網路**:
- 不能建立 bridge networks
- 無法使用 Docker 的進階網路功能

**技術原因**:在 Linux 系統中,操作網路設定需要 root 權限。

##### 2. 適用場景

**較適合的用途**:
- **CI/CD 環境**:Jenkins 等持續整合系統
- **開發環境**:本地開發測試
- **受限環境**:無法取得 root 權限的環境

**較不適合的用途**:
- 需要複雜網路配置的生產環境
- 依賴 Docker 網路功能的微服務架構

### 實作方式

#### 安裝腳本

Docker 提供官方安裝腳本:

```bash
curl -fsSL https://get.docker.com/rootless | sh
```

#### 運作機制

1. **下載與配置**:腳本自動下載 Docker 並配置為使用者服務
2. **Systemd 整合**:仍以 systemd 服務方式運行(非前景模式)
3. **使用者目錄**:Docker Daemon 運行於使用者的家目錄中

#### 啟動與管理

```bash
# 啟動 Docker Rootless
systemctl --user start docker

# 停止服務
systemctl --user stop docker

# 設為開機自動啟動
systemctl --user enable docker
```

### CI 環境的實際應用

#### 典型場景:Jenkins CI Worker

```yaml
# 場景描述
- Jenkins CI Worker 使用 jenkins 使用者身份
- 需要執行 Docker 建置任務
- 但該使用者沒有 root 權限
```

**解決方案**:
1. Jenkins 使用者啟動自己的 Docker Daemon
2. 在使用者家目錄中運行 Docker
3. 執行建置任務
4. 完成後停止 Docker Daemon
5. 全程無需 root 權限

**優勢**:CI/CD 流程不需要 root 權限,降低安全風險。

### 替代方案:繞過網路限制

若需要 Rootless Mode 但又需要網路功能,可能的變通方式:

1. **手動設定網路**:使用其他工具建立網路介面
2. **使用預設網路**:不建立自訂網路,僅使用預設配置
3. **混合架構**:部分功能使用 Docker,網路部分使用其他工具

**評估要點**:需評估實作複雜度與安全性提升的效益平衡。

### 生產環境的適用性評估

#### 不適合生產環境的原因

- **功能受限**:無法使用完整的 Docker 網路功能
- **相容性問題**:許多現有架構依賴 Docker 網路
- **維護複雜度**:需要特殊的配置與管理

#### 適合生產環境的條件

若符合以下條件,可考慮生產環境使用:
- 應用程式不需要自訂網路
- 已有替代的網路解決方案
- 安全性要求極高,值得犧牲便利性

### Windows 容器的安全實踐

#### Windows 與 Linux 的差異

Windows 容器使用不同的安全機制:

**Linux 容器**:
- SELinux
- AppArmor
- Seccomp
- Linux Capabilities

**Windows 容器**:
- Windows 特有的安全機制
- 功能名稱與實作方式不同

#### Windows 容器的安全建議

##### 適用項目

1. **使用 Docker 容器化**:基本隔離效果仍然存在
2. **不以 Administrator 運行**:避免使用 Administrator 帳戶
3. **程式碼掃描**:可掃描 Windows 相關依賴
4. **Content Trust**:可實作映像檔簽章機制

##### 不適用項目

1. **Docker Bench**:無 Windows 版本
2. **User Namespaces**:Windows 不支援此功能
3. **Docker Rootless Mode**:Windows 不支援
4. **Falco**:僅支援 Linux

##### Windows 映像檔掃描的挑戰

需要尋找專門支援 Windows 的掃描工具:
- **.NET Framework 漏洞**:.NET 3.5、.NET Core 2.0 等
- **Windows 套件**:Windows 特有的更新與修補程式
- **相容性**:許多主流掃描工具不支援 Windows 映像檔

### 學習資源

#### Docker Captain 演講

講師頻道上有 Docker Captain 的實際展示:
- Rootless Mode 的詳細操作
- 實際部署案例
- 常見問題排解

#### DockerCon 會議

DockerCon 有專門的技術場次:
- 深入技術細節
- 最佳實踐分享
- 未來發展方向

## 💡 重點摘要

- Rootless Mode 讓 Docker Daemon 以非 root 身份運行,但不支援自訂網路功能
- 最適合 CI/CD 環境,讓非 root 使用者也能執行 Docker 建置任務
- Windows 容器使用不同的安全機制,許多 Linux 工具不適用
- Windows 容器需尋找專門支援 .NET 與 Windows 套件的掃描工具

## 🔑 關鍵字

Rootless Mode, Docker Daemon, Windows Containers, CI/CD Security, Non-root
