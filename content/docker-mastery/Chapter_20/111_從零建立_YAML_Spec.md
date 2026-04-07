# 從零建立 YAML Spec

## 📝 課程概述

本單元深入講解如何從零開始撰寫 Kubernetes YAML 的 `spec` 部分。我們將學習 `kubectl explain` 指令來探索資源的所有可用欄位，以及使用 dry run 和 diff 功能在實際部署前預覽變更，這是生產環境中不可或缺的安全網。

---

## 核心觀念與實作解析

### 使用 kubectl explain 探索 Spec

`spec` 是 YAML 中最複雜的部分，因為每種資源類型的 spec 結構完全不同。`kubectl explain` 是你的最佳幫手。

#### 列出所有可用欄位

```bash
kubectl explain services --recursive
```

這會列出該資源類型的**所有欄位名稱**，方便快速查閱。

> 缺點：不會告訴你每個欄位的用途或是否為必填。

#### 深入查看特定欄位

使用「點號語法」逐層深入：

```bash
kubectl explain services.spec
kubectl explain services.spec.type
```

輸出會包含：

- 欄位描述
- 資料型別（string、boolean、object 等）
- 可接受的值

---

### 點號語法的層層深入

對於複雜資源如 Deployment，可以一路深入：

```bash
# 查看 Deployment 的 spec
kubectl explain deployment.spec

# 查看 template（Pod 模板）
kubectl explain deployment.spec.template

# 查看 Pod 的 spec
kubectl explain deployment.spec.template.spec

# 查看 volumes
kubectl explain deployment.spec.template.spec.volumes

# 查看 NFS volume 的 server 設定
kubectl explain deployment.spec.template.spec.volumes.nfs.server
```

這種方式讓你可以精確找到任何欄位的定義，而不需要翻閱大量文件。

---

### 建立 YAML 的工作流程

老師推薦的流程：

1. **確定 kind 與 apiVersion**：使用 `kubectl api-resources` 和 `kubectl api-versions`
2. **設定 metadata.name**：給資源一個有意義的名稱
3. **探索 spec 結構**：使用 `kubectl explain` 逐層查看
4. **撰寫 YAML**：根據查詢結果填寫欄位

#### 實際範例

假設要建立 Service：

```bash
# 確認 kind 與 API version
kubectl explain service
# 看到 version: v1

# 查看 spec.type 的可接受值
kubectl explain service.spec.type
# 看到: ClusterIP, NodePort, LoadBalancer

# 撰寫 YAML
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  ports:
    - port: 80
```

---

### Dry Run：預覽變更

從 Kubernetes 1.13 開始，支援 **Server-side Dry Run**：

#### Client Dry Run（舊方式）

```bash
kubectl apply -f app.yml --dry-run=client
```

只在本地驗證 YAML 語法，**不會連線到伺服器**。

#### Server Dry Run（推薦）

```bash
kubectl apply -f app.yml --dry-run=server
```

會連線到伺服器：

- 檢查資源是否已存在
- 回報「會建立」還是「會更新」
- **不會實際執行變更**

> 這讓你可以在真正部署前，確認伺服器會如何處理你的 YAML。

---

### kubectl diff：查看具體差異

Server dry run 只告訴你「會不會變更」，但不知道具體改了什麼。`kubectl diff` 可以顯示詳細差異：

```bash
kubectl diff -f app.yml
```

#### 輸出解讀

```diff
- replicas: 3
+ replicas: 2
- labels: {}
+ labels:
+   tier: dmz
```

- `-` 開頭的行：會被移除
- `+` 開頭的行：會被新增

這讓你可以：

1. 確認修改是否如預期
2. 避免意外變更關鍵設定
3. 在 Git commit 前做最後檢查

---

### 建立個人的 YAML 模板庫

老師建議的長期做法：

1. 從官方文件或課程範例開始
2. 逐步建立自己的模板庫
3. 在公司內建立標準化的模板
4. 每個團隊可能有自己習慣的 labels、annotations 設定

這樣一來：

- 不需要每次從頭撰寫
- 確保團隊一致性
- 快速建立新服務的 YAML

---

### API 文件的補充角色

對於習慣閱讀 API 文件的開發者，Kubernetes 官方文件提供了更詳細的說明：

- [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)
- 按 API group 組織（Workloads API、Service API、Storage API 等）
- 包含 swagger 格式的詳細定義

> 注意：API 文件可能顯示舊版本的資訊，命令列的 `kubectl explain` 顯示的是你當前客戶端版本對應的資訊。

---

## 💡 重點摘要

- **使用 `kubectl explain` 搭配點號語法，可以探索任何資源的 spec 結構。**
- **Server-side dry run (`--dry-run=server`) 會連線伺服器驗證，比 client dry run 更準確。**
- **`kubectl diff` 可以顯示 YAML 與現有資源的具體差異，是部署前的安全檢查。**
- **建立個人化的 YAML 模板庫，可以大幅提升日常工作效率。**

---

## 🔑 關鍵字

kubectl explain, dry-run, kubectl diff, spec, template
