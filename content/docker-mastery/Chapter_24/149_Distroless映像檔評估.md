# Distroless Images 精簡映像檔的權衡

## 📝 課程概述

本單元探討 Distroless Images 的概念與實務應用,分析其在映像檔大小與安全性之間的權衡。我們將理解其技術門檻、除錯挑戰,並評估何時該採用此技術,以及如何妥善管理容器中的機密資訊。

## 核心觀念與實作解析

### Distroless Images 的定義

#### 什麼是 Distroless Image?

**Distroless Image** 指的是不包含作業系統套件管理器的映像檔:

**包含的內容**:
- 應用程式本體
- 必要的執行環境

**不包含的內容**:
- 套件管理器(apt、yum、apk)
- Shell 程式
- 系統工具(cURL、vim 等)
- 多餘的系統函式庫

**技術本質**:類似於使用 `FROM scratch` 建置的映像檔,僅包含應用程式的最小運行環境。

#### 術語澄清

- **Distroless Image**:產業通用術語
- **Scratch Image**:從零開始的空映像檔
- **實務上**:兩者概念相近,但 Distroless 通常指 Google 維護的官方專案

### 採用率低的原因分析

#### 原因 1:應用程式依賴性問題

**大多數應用程式需要**:
- 系統函式庫(如 OpenSSL)
- 語言執行環境(如 Python、Ruby)
- 套件依賴(如 pip、npm 套件)
- 系統工具

**適合 Distroless 的語言**:
- **Go**:可編譯成單一靜態二進位檔案
- **C/C++**:靜態連結所有依賴
- **Rust**:支援靜態連結

**不適合的語言**:
- Python、Ruby、PHP 等直譯式語言
- 依賴大量動態連結函式庫的應用

#### 原因 2:團隊技術門檻

採用 Distroless Images 需要團隊具備:

1. **語言能力**:使用支援靜態編譯的語言
2. **建置能力**:理解如何正確建置靜態二進位檔
3. **除錯能力**:熟悉 Linux namespaces 與容器除錯技巧

### 最大的實務挑戰:無法除錯

#### 無 Shell 環境的困境

傳統容器除錯方式:
```bash
docker exec -it container_name /bin/sh
```

**Distroless Image 的問題**:
- 容器內沒有 Shell
- 無法 `exec` 進入容器
- 無法查看檔案系統
- 無法執行診斷工具

#### 解決方案與挑戰

**進階除錯技巧**:
- 使用 sidecar 容器進入同一個 namespace
- 使用 `nsenter` 等工具
- 透過 ephemeral container(Kubernetes)

**技術要求**:
- 理解 Linux namespaces 運作機制
- 熟悉底層除錯工具
- 非一般 Docker 指令所能解決

### 安全性效益的理性評估

#### 理論上的安全優勢

移除多餘工具的好處:
- 減少攻擊面
- 移除潛在的漏洞點
- 攻擊者無法使用系統工具

**情境舉例**:
若容器內有 cURL 但應用程式不用它:
- cURL 存在漏洞
- 攻擊者需先突破應用程式
- 才能利用 cURL 進一步攻擊

#### 實務上的安全疑慮

##### 靜態連結的盲點

若應用程式靜態連結 OpenSSL:
- 漏洞仍存在於二進位檔中
- 只是從獨立檔案變成嵌入應用程式
- **掃描工具無法偵測**:大多數掃描器無法識別靜態連結的漏洞

```go
// Go 靜態編譯範例
// OpenSSL 程式碼被編譯進單一 binary
// 漏洞掃描器可能無法檢測
```

##### 現實世界的威脅

講師觀察到的真實問題:
- Docker Daemon 暴露在網路上
- Kubernetes 缺乏認證機制
- 配置錯誤導致服務暴露

**這些才是高風險威脅**,而非理論上的映像檔最小化問題。

### 實施建議與優先順序

#### 何時該考慮 Distroless Images?

**適合場景**:
- 團隊已是 Go 專家
- 熟悉 Linux namespaces
- 有完善的除錯工具鏈
- 安全性要求極高

**不適合場景**:
- 容器化初期專案
- 團隊尚在學習容器技術
- 使用 Python、Ruby 等語言
- 時間壓力大的專案

#### 優先順序建議

講師的安全實踐優先級:

**第一優先**(實際威脅):
1. 正確配置 Docker 與 Kubernetes
2. 使用 Secrets 管理機密
3. 映像檔漏洞掃描
4. 網路隔離與認證

**第二優先**(進階實踐):
5. User Namespaces
6. 自訂安全設定檔
7. Runtime 行為監控

**最後考慮**(理論效益):
8. Distroless Images

### 機密資訊管理的最佳實踐

#### 問題:Docker Secrets 存在記憶體中

**學員提問**:
> "Docker Secrets 只是將密碼放在容器的 RAM 檔案中,若駭客入侵容器,仍可讀取該檔案。"

#### 講師回應:安全性的現實考量

##### 基本原則

**無法完全避免的風險**:
- 應用程式必須在某處存取機密
- 解密後的資料必然存在記憶體中
- 若攻擊者取得 root 權限,即可存取記憶體

##### 防禦措施層級

**第一層:不以 root 運行**
```dockerfile
USER appuser
```
- 攻擊者無法使用系統工具
- 無法讀取其他程序的記憶體
- 限制攻擊者的行動能力

**第二層:使用 Secrets 機制**

**Docker Secrets 與 Kubernetes Secrets 的優勢**:
- 檔案存在於 RAM(t tmpfs)
- 非永久儲存於磁碟
- 可設定檔案權限
- 僅特定使用者可讀取

**實作範例**:
```yaml
# Kubernetes Secret
apiVersion: v1
kind: Secret
metadata:
  name: db-password
type: Opaque
data:
  password: <base64-encoded>
```

**第三層:一次性密碼(Vault)**

**HashiCorp Vault 等解決方案**:
- 動態產生密碼
- 使用後立即失效
- 需整合整個系統架構

**限制**:
- 並非所有服務支援動態密碼
- 需要大量架構設計
- 實作成本高

#### 現實世界的最佳實踐

**大多數組織的現況**:
- 處理既有應用程式
- 需要快速導入容器化
- 無法重構整個架構

**實用建議**:
1. **優先使用 Secrets**:已比 99% 的組織更安全
2. **加密靜態儲存**:Kubernetes 與 Swarm 都支援加密儲存
3. **避免環境變數**:環境變數容易洩漏,應優先使用檔案型 Secrets
4. **權限控管**:設定檔案權限,僅允許應用程式使用者讀取

## 💡 重點摘要

- Distroless Images 適合 Go 等可靜態編譯的語言,但會增加除錯難度
- 理論上的安全效益有限,靜態連結的漏洞仍存在且難以掃描
- 應優先處理實際威脅(配置錯誤、認證缺失),而非追求理論上的最小化
- 使用 Docker/Kubernetes Secrets 已是相對安全的做法,無需過度擔心記憶體洩漏風險

## 🔑 關鍵字

Distroless Images, Scratch Image, Static Binary, Secrets Management, Container Debugging
