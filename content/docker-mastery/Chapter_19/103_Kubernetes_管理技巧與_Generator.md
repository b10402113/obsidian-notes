# Kubernetes 管理技巧與 Generator 機制

## 📝 課程概述

本單元介紹 Kubernetes 的管理技巧與 Generator 機制。Generator 是指令背後的模板引擎，會自動填補你未指定的設定值。理解 Generator 如何運作，有助於我們從指令式操作過渡到宣告式配置。

---

## 核心觀念與實作解析

### Kubernetes 的不帶意見設計

Kubernetes 與 Swarm 的關鍵差異：

| 面向 | Swarm | Kubernetes |
| --- | --- | --- |
| **工作流程** | 有明確的建議做法 | 不帶意見，多種方式可行 |
| **學習曲線** | 較平緩 | 需要更多時間理解選項 |
| **彈性** | 較受限 | 極大彈性，適應各種工作流程 |

> 彈性的代價是：你需要花更多時間理解「什麼是最佳實踐」。

---

### 什麼是 Generator？

Generator 是指令背後的**模板引擎**：

- 當你執行 `kubectl run` 或 `kubectl create` 時
- Generator 會根據你的參數，產生完整的 YAML spec
- 未指定的欄位會填入預設值

```
kubectl create deployment test --image nginx
                ↓
         Generator 運作
                ↓
    產生完整 YAML spec（包含預設值）
                ↓
           送至 API Server
```

#### Generator 的特性

1. **每種資源類型有專屬 Generator**：Deployment、Service、Job、CronJob 各有不同
2. **有版本區分**：beta 版本可能隨 Kubernetes 版本升級為 v1
3. **自動填補預設值**：只寫 10-15 行核心設定，其餘自動補足

---

### --dry-run 與 -o yaml：預覽 Generator 輸出

這是學習 YAML spec 的最佳工具：

```bash
kubectl create deployment test --image nginx --dry-run=client -o yaml
```

- `--dry-run=client`：不在叢集上執行，只模擬
- `-o yaml`：輸出 Generator 產生的 YAML

#### 輸出範例：Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

> 這就是 Generator 為你產生的模板，你可以用來學習 YAML 結構。

---

### 不同資源類型的 Generator 差異

#### Deployment Generator

結構較複雜，因為包含多層嵌套：

```
Deployment
    └── ReplicaSet (template)
            └── Pod (spec)
```

#### Job Generator

```bash
kubectl create job test --image nginx --dry-run=client -o yaml
```

特點：
- 結構較簡單（沒有 ReplicaSet 層級）
- `restartPolicy` 預設為 `Never`
- 適合一次性任務

#### Service Generator

```bash
# 需要先建立 Deployment
kubectl create deployment test --image nginx

# 再產生 Service 的 YAML
kubectl expose deployment test --port 80 --dry-run=client -o yaml
```

Service 的 spec 主要包含：
- ports（port 與 targetPort）
- selector
- type

---

### Generator 的實務應用

| 用途 | 說明 |
| --- | --- |
| **學習 YAML 結構** | 使用 `--dry-run -o yaml` 查看完整 spec |
| **產生模板起點** | 將輸出存為檔案，再修改客製化設定 |
| **除錯** | 確認指令會產生什麼樣的資源定義 |

> Generator 產生的模板是「最小可用」的設定，實務上通常需要補充更多細節。

---

## 💡 重點摘要

- **Generator 是指令背後的模板引擎，自動填補未指定的預設值。**
- **使用 `--dry-run=client -o yaml` 可預覽 Generator 產生的完整 YAML。**
- **不同資源類型有不同的 Generator，結構與預設值各有差異。**
- **Job 的 restartPolicy 預設為 Never，Deployment 則為 Always。**
- **Generator 產生的模板是學習 YAML spec 的最佳起點。**

---

## 🔑 關鍵字

Generator, dry-run, YAML, Spec, Template, Default Values
