# Kubernetes YAML 配置結構解析

## 📝 課程概述

本單元深入探討 Kubernetes YAML 檔案的結構與組成要素。相比 Docker Compose，Kubernetes YAML 確實更複雜，但這也帶來了更大的彈性。我們將學習 YAML 的四個必要根元素，以及如何查詢不同資源類型的 API version 與 kind。

---

## 核心觀念與實作解析

### YAML vs JSON

Kubernetes 支援兩種格式：

| 格式 | 優點 | 適合對象 |
|------|------|----------|
| **YAML** | 人類可讀性高、容易編輯 | 人類撰寫 |
| **JSON** | 機器解析效率高 | 電腦處理 |

實際運作時，Kubernetes 會將 YAML 轉換為 JSON 再處理。我們作為人類，標準做法是撰寫 YAML。

---

### YAML 的四個必要根元素

每個 Kubernetes YAML 檔案（稱為 **Manifest**）都必須包含四個根元素：

```yaml
apiVersion: apps/v1        # 1. API 版本
kind: Deployment           # 2. 資源類型
metadata:                  # 3. 元資料
  name: my-app
spec:                      # 4. 規格定義
  replicas: 3
  # ...
```

#### 1. apiVersion

指定使用的 API 版本。不同資源類型可能對應不同的 API group：

- `v1`：核心 API（如 Pod、Service）
- `apps/v1`：應用相關 API（如 Deployment、ReplicaSet）
- `extensions/v1beta1`：已棄用的舊 API

#### 2. kind

資源類型名稱，常見的有：

- `Pod`
- `Deployment`
- `Service`
- `ConfigMap`
- `Secret`
- `Ingress`

#### 3. metadata

必要欄位是 `name`，用於識別資源：

```yaml
metadata:
  name: my-deployment
  labels:
    app: nginx
```

#### 4. spec

這是最關鍵的部分，**不同資源類型的 spec 結構完全不同**：

- Pod 的 spec 定義 containers
- Deployment 的 spec 定義 replicas 與 template
- Service 的 spec 定義 ports 與 selector

---

### 查詢可用的資源類型

使用 `kubectl api-resources` 查看叢集中所有可用的資源類型：

```bash
kubectl api-resources
```

輸出包含幾個重要欄位：

| 欄位 | 說明 |
|------|------|
| **NAME** | 資源名稱 |
| **KIND** | YAML 中使用的 kind 值 |
| **APIGROUP** | API 群組名稱 |

> 注意：同一種資源可能出現多次，對應不同的 API group（如新舊版本）。

---

### 查詢 API 版本

使用 `kubectl api-versions` 查看所有 API 版本：

```bash
kubectl api-versions
```

這可以幫助你確認：

- 目前支援哪些 API 版本
- 哪些版本已被棄用
- `kind` 與 `apiVersion` 的對應關係

---

### API 版本的演進範例

以 `Deployment` 為例，API 版本的演變：

```
extensions/v1beta1  →  apps/v1beta1  →  apps/v1beta2  →  apps/v1
      (已棄用)           (已棄用)           (已棄用)          (目前穩定版)
```

**為什麼這很重要？**

當你使用舊的 YAML 檔案部署到新的 Kubernetes 叢集時：

1. 叢集會回報「不支援此 API 版本」錯誤
2. 你需要查詢文件或 GitHub issues 找出正確的新版本
3. 更新 YAML 中的 `apiVersion`

---

### 多資源單一檔案

使用 `---` 分隔多個資源定義：

```yaml
# 第一個資源：Service
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: nginx
  ports:
    - port: 80

---
# 第二個資源：Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    # ...
```

這種方式適合將相關資源放在一起管理。

---

### CRD 自訂資源

Kubernetes 的獨特之處在於可以透過 **CRD (Custom Resource Definition)** 擴充資源類型：

- 第三方工具可以新增自訂的資源類型
- 這些新資源也會出現在 `kubectl api-resources` 列表中
- YAML 格式同樣支援這些自訂資源

這是 Kubernetes 比 Docker Swarm 更具擴充性的關鍵特性。

---

## 💡 重點摘要

- **每個 Kubernetes YAML 必須包含四個根元素：apiVersion、kind、metadata、spec。**
- **不同資源類型對應不同的 API group 與版本，需要透過指令查詢確認。**
- **API 版本會隨時間演進，舊版本會被棄用，需定期更新 YAML。**
- **使用 `kubectl api-resources` 和 `kubectl api-versions` 查詢可用的資源與版本。**

---

## 🔑 關鍵字

apiVersion, kind, metadata, spec, CRD
