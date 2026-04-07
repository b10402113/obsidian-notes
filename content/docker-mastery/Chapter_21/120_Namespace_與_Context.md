# Namespace 與 Context

## 📝 課程概述

本單元介紹 Kubernetes 中的 **Namespace** 與 **Context** 機制，這是在多團隊、多環境場景下管理叢集的關鍵概念。同時，講師也分享了對 Kubernetes 未來發展的看法，幫助學員掌握產業趨勢與技術走向。

---

## 核心觀念與實作解析

### Namespace：虛擬叢集

**Namespace** 讓你可以在單一叢集中建立多個「虛擬叢集」或「虛擬視圖」。

#### 核心功能

- **視圖過濾**：不同 Namespace 下的資源彼此隔離，`kubectl get` 只會看到當前 Namespace 的資源
- **網路隔離**：搭配 Network Policy，可以控制不同 Namespace 間的通訊

> **重要區別**：Kubernetes 的 Namespace 與 Docker/Linux 的 Namespace 是完全不同的概念。前者是 Kubernetes 的資源隔離機制，後者是 Linux 核心層級的容器隔離技術。

#### 預設 Namespace

執行 `kubectl get namespaces` 會看到幾個預設 Namespace：

| Namespace | 用途 |
| --- | --- |
| **default** | 預設命名空間，你的應用通常部署在這裡 |
| **kube-system** | Kubernetes 系統元件運行於此 |
| **kube-public** | 公開資源 |
| **kube-node-lease** | 節點心跳資訊 |

當你執行 `kubectl get all` 時，預設只會顯示 `default` Namespace 的資源，這隱藏了系統運作的複雜性。

---

### Context：叢集連線設定

**Context** 定義了 `kubectl` 連線到哪個叢集、使用哪個帳號、以及操作哪個 Namespace。

#### Context 的三個組成要素

1. **Cluster**：目標 Kubernetes 叢集
2. **User**：認證身分（決定你的權限範圍）
3. **Namespace**：預設操作的命名空間

#### 設定檔位置

所有 Context 設定儲存在 `~/.kube/config` 這個 YAML 檔案中：

```yaml
contexts:
- context:
    cluster: docker-desktop
    namespace: default
    user: docker-desktop
  name: docker-desktop
```

> **安全提醒**：這個檔案包含認證憑證，請勿分享或提交到版本控制系統。

#### 常用指令

```bash
# 查看所有 Context
kubectl config get-contexts

# 切換 Context
kubectl config use-context minikube

# 查看當前 Context
kubectl config current-context
```

#### 實用技巧

有多個叢集時，你可以在 shell prompt 中顯示當前 Context（類似 Git branch 顯示），避免操作錯誤的叢集。市面上有許多 shell plugin 可以達成這個效果。

---

### Kubernetes 的未來趨勢

#### 1. 讓基礎設施變得「無聊」

Kubernetes 創始團隊的目標是讓 Kubernetes 成為穩定、可靠、不需要太多關注的基礎設施。重點轉向：

- **可靠性**（Reliability）
- **穩定性**（Stability）
- **安全性**（Security）

#### 2. 安全性改進

2019 年 Kubernetes 進行了完整的第三方安全稽核，並公開了所有發現與修復計畫。這展現了開源專案的成熟度與透明度。

#### 3. 舊功能的淘汰

- 非 CSI 的儲存 Plugin 將逐步被移除
- `kubectl run` 將簡化為單純的 Pod 建立命令

#### 4. GitOps 與 Declarative Workflow

**GitOps** 是一種新興的運維方法論：

- 將基礎設施配置存放在 Git 中
- 透過 Git commit 來驅動基礎設施變更
- 所有變更都有版本控制與審計軌跡

Kubernetes 的功能發展越來越支援這種聲明式工作流。

#### 5. Windows Server 支援

Kubernetes 在 Windows Server 2019 上開始提供官方支援，但仍落後 Docker/Swarm 多年的成熟度。這將是未來的發展重點。

#### 6. 邊緣運算與輕量化

專案如 **K3s**（Rancher 出品）致力於：

- 精簡 Kubernetes 二進位檔
- 移除不必要的遺留程式碼
- 適用於 IoT、ARM、Raspberry Pi 等資源受限環境

---

### 相關專案推薦

| 專案 | 用途 |
| --- | --- |
| **Knative** | Serverless 框架（CNCF 官方專案） |
| **K3s** | 輕量化 Kubernetes 發行版 |
| **K3os** | 內建 K3s 的輕量作業系統 |
| **Service Mesh（Istio/Envoy）** | 微服務通訊層，提供監控、安全與路由能力 |

> **Service Mesh** 是近年來的新概念，專門解決微服務架構下「服務間通訊」的複雜性問題。它在所有服務間加入一層 Proxy，提供統一的可觀測性、安全性與流量控制。

---

## 💡 重點摘要

- **Namespace 是 Kubernetes 的資源隔離機制，可建立多個虛擬叢集；Context 則定義 kubectl 連線的叢集、帳號與 Namespace。**
- **kubectl 的設定檔位於 ~/.kube/config，包含所有叢集連線資訊與認證憑證，應妥善保護。**
- **Kubernetes 的發展方向是讓基礎設施變得穩定可靠，GitOps 與聲明式工作流是重要趨勢。**
- **Service Mesh 是解決微服務通訊複雜性的新興技術，提供統一的可觀測性與安全性。**

---

## 🔑 關鍵字

Namespace, Context, kubectl, GitOps, Service Mesh
