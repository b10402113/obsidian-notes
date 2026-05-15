# Kubernetes Deployment 測驗

**科目：** Kubernetes 入門課程
**主題：** Deployment：讓應用永不宕機
**難度：** 混合（基礎 40%、中等 40%、進階 20%）
**題數：** 15 題
**建議時間：** 20-25 分鐘

---

## 第一部分：選擇題（每題 4 分，共 20 分）

**Q1.** 在 Kubernetes 中，Deployment API 對象主要用於管理哪種類型的業務？

A) 離線業務（如批處理任務）
B) 在線業務（如持續運行的服務）
C) 定時任務
D) 單次執行作業

<details>
<summary>答案</summary>
<b>B) 在線業務（如持續運行的服務）</b>

<b>解析：</b>Deployment 專門用於管理在線業務，這類業務需要持續運行並對外提供服務。離線業務由 Job 和 CronJob 管理。Deployment 能夠確保應用始終保持運行狀態，實現「永不宕機」的目標。
</details>

---

**Q2.** 在 Deployment 的 YAML 定義中，<code>replicas</code> 字段的作用是什麼？

A) 定義 Pod 的重啟策略
B) 指定要運行的 Pod 實例數量
C) 設置容器的副本數量
D) 控制滾動更新的速度

<details>
<summary>答案</summary>
<b>B) 指定要運行的 Pod 實例數量</b>

<b>解析：</b>replicas 字段定義了 Kubernetes 集群中需要運行的 Pod 實例數量，也就是「副本數量」。Deployment 會根據這個字段自動維護 Pod 的數量，確保實際運行的 Pod 數量與期望狀態一致。
</details>

---

**Q3.** 為什麼 Deployment 需要使用 <code>selector.matchLabels</code> 字段？

A) 為了自動生成 Pod 名稱
B) 為了篩選出被 Deployment 管理的 Pod 對象
C) 為了設置 Pod 的資源限制
D) 為了定義 Pod 的運行節點

<details>
<summary>答案</summary>
<b>B) 為了篩選出被 Deployment 管理的 Pod 對象</b>

<b>解析：</b>selector.matchLabels 字段定義了篩選規則，用於識別哪些 Pod 應該被 Deployment 管理。這是因為 Deployment 和 Pod 是鬆散的組合關係，Deployment 通過標籤（labels）來「找到」它管理的 Pod，而不是強綁定。這個字段必須與 template.metadata.labels 完全一致。
</details>

---

**Q4.** 當使用 <code>kubectl get deploy</code> 查看 Deployment 狀態時，READY 欄位顯示「2/2」表示什麼？

A) 有 2 個 Pod 正在創建中
B) 當前有 2 個 Pod 運行，期望有 2 個 Pod 運行
C) 有 2 個節點可用
D) 已更新 2 個 Pod

<details>
<summary>答案</summary>
<b>B) 當前有 2 個 Pod 運行，期望有 2 個 Pod 運行</b>

<b>解析：</b>READY 欄位的格式是「當前數量/期望數量」。「2/2」表示當前有 2 個 Pod 正在運行，而期望的 Pod 數量也是 2 個，說明 Deployment 已經達到了期望狀態。如果顯示「1/2」則表示還有 1 個 Pod 尚未就緒。
</details>

---

**Q5.** 如果不小心刪除了被 Deployment 管理的 Pod，會發生什麼？

A) Pod 會永久消失，需要手動重建
B) Deployment 會自動創建新的 Pod 來替換被刪除的 Pod
C) Deployment 會報錯並停止服務
D) 其他 Pod 會接管被刪除 Pod 的工作

<details>
<summary>答案</summary>
<b>B) Deployment 會自動創建新的 Pod 來替換被刪除的 Pod</b>

<b>解析：</b>Deployment 會持續監控 Pod 的運行狀態，確保實際運行的 Pod 數量與期望狀態（replicas 字段定義的數量）一致。當 Pod 意外消失時，Deployment 會自動創建新的 Pod 來替換，這就是 Deployment 實現「永不宕機」的核心機制。
</details>

---

## 第二部分：是非題（每題 3 分，共 15 分）

**Q6.** Deployment 和 Job 一樣，採用強綁定的方式管理 Pod 對象。_(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b>

<b>解析：</b>Deployment 和 Job 的管理方式不同。Job 採用強綁定方式，Pod 與 Job 緊密關聯。而 Deployment 採用鬆散的組合關係，通過 labels 和 selector 來識別和管理 Pod。這種設計允許其他 API 對象（如 Service）也能引用這些 Pod，提供了更大的靈活性。
</details>

---

**Q7.** 在 Deployment 的 YAML 文件中，<code>selector.matchLabels</code> 和 <code>template.metadata.labels</code> 必須完全相同。_(對/錯)_

<details>
<summary>答案</summary>
<b>對</b>

<b>解析：</b>這兩個字段必須完全一致，否則 Deployment 無法找到它應該管理的 Pod，apiserver 也會拒絕創建並報告 YAML 格式校驗錯誤。selector.matchLabels 定義了篩選規則，而 template.metadata.labels 為創建的 Pod 打上標籤，兩者必須匹配才能建立管理關係。
</details>

---

**Q8.** <code>kubectl scale</code> 命令是聲明式操作，適合用於長期維護應用。_(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b>

<b>解析：</b>kubectl scale 是命令式操作，只適合作為臨時的擴容或縮容措施。如果應用需要長時間保持特定的 Pod 數量，應該編輯 Deployment 的 YAML 文件，修改 replicas 字段，然後使用聲明式的 kubectl apply 來更新對象狀態。這樣可以確保配置的一致性和可追溯性。
</details>

---

**Q9.** Deployment 可以用於管理有狀態的應用，如數據庫。_(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b>

<b>解析：</b>Deployment 主要用於部署無狀態應用。無狀態應用沒有特殊的啟動順序要求，所有 Pod 都被視為相同的服務實例。對於有狀態應用（如數據庫），需要使用 StatefulSet 等 API 對象來管理，這類應用需要穩定的網絡標識、持久化存儲和有序的部署/擴縮容。
</details>

---

**Q10.** 即使只運行一個 Pod，也應該使用 Deployment 而不是直接創建裸 Pod。_(對/錯)_

<details>
<summary>答案</summary>
<b>對</b>

<b>解析：</b>即使 replicas 設為 1，使用 Deployment 也能獲得自動維護、故障恢復等能力。Deployment 會確保應用始終在線，如果 Pod 意外終止或被刪除，Deployment 會自動重建。直接創建裸 Pod 缺乏這種保護機制，不符合 Kubernetes 的最佳實踐。
</details>

---

## 第三部分：填充題（每題 6 分，共 12 分）

**Q11.** 在 Deployment 中，**\_\_\_\_** 字段定義了 Pod 的「期望數量」，而 **\_\_\_\_** 字段定義了基於標籤篩選 Pod 的規則。

<details>
<summary>答案</summary>
<b>replicas, selector</b>

<b>解析：</b>replicas 字段指定要運行的 Pod 實例數量，Kubernetes 會自動維護這個數量。selector 字段（特別是其 matchLabels 子字段）定義了篩選規則，用於識別哪些 Pod 屬於該 Deployment 管理。這兩個字段共同實現了 Deployment 的核心功能。
</details>

---

**Q12.** 使用 <code>kubectl scale --replicas=5 deploy ngx-dep</code> 命令可以將應用擴容到 5 個實例，但這屬於 **\_\_\_\_** 操作；如果要長期維護這個數量，應該修改 Deployment 的 YAML 文件並使用 **\_\_\_\_** 命令。

<details>
<summary>答案</summary>
<b>命令式（imperative）, kubectl apply</b>

<b>解析：</b>kubectl scale 是命令式操作，直接修改當前狀態但不改變配置文件。聲明式操作 kubectl apply 則是根據 YAML 配置文件來維護期望狀態，更適合長期維護和版本控制。生產環境建議使用聲明式方法管理資源。
</details>

---

## 第四部分：簡答題（每題 10 分，共 20 分）

**Q13.** 請解釋為什麼 Deployment 需要同時設置 <code>selector.matchLabels</code> 和 <code>template.metadata.labels</code>，且兩者必須一致。

<details>
<summary>答案</summary>
<b>參考答案：</b>

Deployment 需要同時設置這兩個字段且保持一致，原因如下：

1. **鬆散組合關係**：與 Job 的強綁定不同，Deployment 和 Pod 採用鬆散的組合關係。Deployment 不直接「持有」Pod，而是通過標籤選擇器來識別和管理 Pod。

2. **篩選機制**：
   - <code>selector.matchLabels</code> 定義了「篩選規則」，告訴 Deployment 應該管理哪些 Pod
   - <code>template.metadata.labels</code> 為新創建的 Pod 打上「標籤」

3. **必須一致**：兩者必須完全相同，否則 Deployment 無法找到它創建的 Pod，apiserver 也會拒絕創建並報錯。

4. **靈活性優勢**：這種設計允許其他 API 對象（如 Service）也能通過標籤引用這些 Pod，提供了更大的靈活性和解耦能力。

<b>關鍵點：</b>
- Deployment 通過標籤建立與 Pod 的管理關係
- 實現了「弱引用」而非強綁定
- 支持 Pod 被多個對象引用管理
</details>

---

**Q14.** 當 Deployment 管理的 Pod 被誤刪或所在的節點發生故障時，Deployment 如何確保應用「永不宕機」？請描述整個自動恢復流程。

<details>
<summary>答案</summary>
<b>參考答案：</b>

Deployment 的自動恢復流程：

1. **持續監控**：Deployment 控制器會持續監控集群中運行的 Pod 數量和狀態，通過 selector 字段定義的標籤選擇器識別屬於自己的 Pod。

2. **狀態比較**：控制器將實際運行的 Pod 數量與 YAML 中定義的 replicas 字段（期望數量）進行比較。

3. **檢測差異**：當 Pod 被刪除或節點故障導致 Pod 消失時，實際數量小於期望數量，Deployment 檢測到這種差異。

4. **自動重建**：Deployment 通過 apiserver、scheduler 等核心組件：
   - 選擇新的健康節點
   - 根據 Pod 模板創建新的 Pod
   - 直到 Pod 數量與期望狀態一致

5. **持續維護**：這個過程完全自動化，無需人工干預，確保應用始終有足夠的實例在運行。

<b>關鍵點：</b>
- 基於「期望狀態」的設計理念
- 自動化的監控和恢復機制
- 全程無需人工介入
- READY 欄位顯示當前/期望數量，方便監控狀態
</details>

---

## 第五部分：配對題（每題 8 分，共 16 分）

**Q15.** 請將以下 kubectl 命令與其對應的功能進行配對：

| 命令 | 功能 |
|------|------|
| 1. <code>kubectl apply -f deploy.yml</code> | A. 查看帶有特定標籤的 Pod |
| 2. <code>kubectl get deploy</code> | B. 創建或更新 Deployment |
| 3. <code>kubectl scale --replicas=5 deploy ngx-dep</code> | C. 擴容或縮容 Pod 數量 |
| 4. <code>kubectl get pod -l app=nginx</code> | D. 查看 Deployment 狀態 |

<details>
<summary>答案</summary>
1-B, 2-D, 3-C, 4-A

<b>解析：</b>
- <b>kubectl apply -f deploy.yml</b>：聲明式操作，根據 YAML 文件創建或更新 Deployment
- <b>kubectl get deploy</b>：查看 Deployment 的狀態信息，包括 READY、UP-TO-DATE、AVAILABLE 等欄位
- <b>kubectl scale --replicas=5 deploy ngx-dep</b>：命令式操作，快速調整 Pod 的副本數量為 5
- <b>kubectl get pod -l app=nginx</b>：使用標籤選擇器篩選並查看特定標籤的 Pod，-l 參數用於標籤過濾

<b>補充說明：</b>
- <code>-l</code> 參數支持多種表達式：==、!=、in、notin
- 例如：<code>kubectl get pod -l 'app in (ngx, nginx, ngx-dep)'</code>
- 標籤篩選功能與 Deployment 的 selector 機制相同
</details>

---

## 第六部分：應用題（17 分）

**Q16.** 假設你正在部署一個 Web 應用，需要創建一個名為 <code>web-app</code> 的 Deployment，使用 <code>nginx:latest</code> 鏡像，並保持 3 個 Pod 副本運行。

請寫出完整的 Deployment YAML 配置文件，並解釋以下問題：

1. 如果在生產環境中需要將副本數增加到 5 個，應該採用哪種方式？為什麼？
2. 如何驗證 Deployment 已經成功創建並正常運行？

<details>
<summary>答案</summary>
<b>YAML 配置文件：</b>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

<b>問題解答：</b>

<b>1. 生產環境擴容方法：</b>

<b>推薦方式：</b>修改 YAML 文件中的 replicas 字段為 5，然後執行 <code>kubectl apply -f deploy.yml</code>

<b>原因：</b>
- 聲明式操作，配置文件即為「真實來源」（Single Source of Truth）
- 便於版本控制和追蹤變更歷史
- 配置可追溯、可重現
- 符合 GitOps 實踐，適合 CI/CD 流程
- 團隊協作時避免配置漂移

<b>不推薦：</b>使用 <code>kubectl scale --replicas=5 deploy web-app</code>（命令式操作）
- 臨時措施，不改變配置文件
- 無法追蹤變更歷史
- 容易造成配置不一致

<b>2. 驗證方法：</b>

<b>步驟一：檢查 Deployment 狀態</b>
```bash
kubectl get deploy web-app
```
查看：
- READY 欄位應顯示「3/3」（當前/期望）
- UP-TO-DATE 應為 3
- AVAILABLE 應為 3

<b>步驟二：檢查 Pod 狀態</b>
```bash
kubectl get pod -l app=web-app
```
查看：
- 應有 3 個 Pod 處於 Running 狀態
- Pod 名稱格式：<code>web-app-[hash]-[random]</code>
- READY 欄位應顯示「1/1」

<b>步驟三：檢查詳細信息</b>
```bash
kubectl describe deploy web-app
```
查看：
- 事件日誌確認 Pod 創建成功
- 副本數統計信息

<b>步驟四：測試服務可用性（可選）</b>
```bash
kubectl port-forward deploy/web-app 8080:80
curl http://localhost:8080
```
</details>

---

## 測驗總結

本測驗涵蓋了 Kubernetes Deployment 的核心概念和實際操作：

**重點知識：**
1. Deployment 的作用：管理在線業務，確保應用永不宕機
2. 關鍵字段：replicas（副本數量）、selector（標籤選擇器）
3. labels 機制：鬆散組合關係，支持多對象引用
4. 自動恢復：基於期望狀態的自動化維護
5. 操作命令：kubectl apply（聲明式）、kubectl scale（命令式）

**最佳實踐：**
- 始終使用 Deployment 管理應用，即使只需一個 Pod
- 生產環境優先使用聲明式操作（kubectl apply）
- 確保 selector.matchLabels 與 template.metadata.labels 一致
- 使用標籤進行資源篩選和管理

---

*生成來源：Kubernetes 入門課程 - 第 18 課：Deployment：讓應用永不宕機*