# 宣告式 Kubernetes YAML 單元介紹

## 📝 課程概述

本單元正式進入 Kubernetes 的 Production 實戰模式。我們將從「指令式」的操作方式，轉向「宣告式」的 YAML 配置管理。這種方式符合 DevOps 的 Infrastructure as Code 與 GitOps 精神，是管理 Production 環境的推薦做法。

---

## 核心觀念與實作解析

### 三種 Kubernetes 管理方式回顧

我們之前討論過三種管理 Kubernetes workloads 的方式：

1. **Imperative Commands**：直接用 `kubectl run`、`kubectl expose` 等指令
2. **Imperative Objects**：使用 `kubectl create`、`kubectl edit`、`kubectl replace`
3. **Declarative Objects**：使用 `kubectl apply -f` 配合 YAML 檔案

本單元將專注於**第三種方式**，這是最符合 DevOps 精神的做法。

---

### 為什麼選擇宣告式方法？

宣告式方法的核心優勢在於：

| 特性 | 說明 |
|------|------|
| **Infrastructure as Code** | 所有配置都以程式碼形式存在，可版本控制 |
| **GitOps 友善** | YAML 存放在 Git repository，用 git commit 管理變更 |
| **可重現性** | 同一份 YAML 可以在任何環境重現相同結果 |
| **單一指令** | 建立、更新、修改都用同一個 `kubectl apply` 指令 |

> 老師強調：`kubectl apply` 就是你要學會的**唯一指令**，無論是建立、更新還是修改配置，都用它。

---

### kubectl apply 的使用方式

`kubectl apply -f` 支援多種輸入來源：

#### 1. 單一 YAML 檔案

```bash
kubectl apply -f deployment.yaml
```

#### 2. 整個目錄

```bash
kubectl apply -f ./k8s/
```

會載入目錄下所有的 YAML 檔案。

#### 3. URL 遠端檔案

```bash
kubectl apply -f https://example.com/pod.yaml
```

> **安全提醒**：直接從網路執行 YAML 檔案就像 `curl | sh` 一樣危險，請務必先檢查檔案內容！

#### 4. 多資源單一檔案

一個 YAML 檔案可以包含多個資源定義，使用 `---` 分隔：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  # ...
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  # ...
```

---

### 為什麼不教 kubectl create/edit/replace？

這些指令雖然可用，但不符合 DevOps 的最佳實踐：

- 沒有完整的 audit trail
- 不容易追蹤變更歷史
- 難以實現自動化部署

老師建議直接跳過這些指令，專注於 `kubectl apply` 的宣告式方法。

---

## 💡 重點摘要

- **宣告式方法使用 `kubectl apply -f` 配合 YAML 檔案，是管理 Production Kubernetes 的推薦方式。**
- **同一個 `kubectl apply` 指令用於建立、更新和修改資源，類似 Docker Swarm 的 stack deploy。**
- **YAML 檔案可以存放在 Git 中，實現 Infrastructure as Code 與 GitOps 工作流程。**
- **一個 YAML 檔案可以包含多個資源定義，使用 `---` 分隔不同資源。**

---

## 🔑 關鍵字

kubectl apply, YAML, Declarative, GitOps, Infrastructure as Code
