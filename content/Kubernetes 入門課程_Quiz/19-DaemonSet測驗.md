# DaemonSet 測驗：忠實可靠的看門狗

**科目：** Kubernetes 入門課程
**主題：** DaemonSet
**難度：** 混合（簡易 40%、中等 40%、困難 20%）
**題數：** 18 題
**建議時間：** 25 分鐘

---

## 第一部分：選擇題（每題 5 分，共 25 分）

**Q1.** 下列哪種應用場景最適合使用 DaemonSet 進行部署？

A) Web 前端應用，需要根據流量動態擴容縮容
B) 日誌收集應用 Fluentd，需要在每個節點上收集容器日誌
C) 資料庫 MySQL，需要穩定的高可用架構
D) API 服務，需要負載平衡器分配請求

<details>
<summary>答案</summary>
<b>B) 日誌收集應用 Fluentd，需要在每個節點上收集容器日誌</b>

DaemonSet 的目標是在集群的每個節點上運行且僅運行一個 Pod，這對於需要在每個節點上運行的守護進程類應用（如日誌收集、監控代理、網絡代理）非常適合。Web 前端、資料庫和 API 服務通常使用 Deployment 部署更為合適。
</details>

---

**Q2.** DaemonSet 的 YAML 描述文件與 Deployment 相比，主要的差異是什麼？

A) DaemonSet 沒有 metadata 欄位
B) DaemonSet 沒有 spec.selector 欄位
C) DaemonSet 沒有 spec.replicas 欄位
D) DaemonSet 沒有 spec.template 欄位

<details>
<summary>答案</summary>
<b>C) DaemonSet 沒有 spec.replicas 欄位</b>

DaemonSet 不需要 replicas 欄位，因為它的目標是在每個節點上運行一個 Pod 實例，Pod 的數量由節點數量決定，而不是手動設定副本數。DaemonSet 仍然有 selector 和 template 欄位，這些與 Deployment 相似。
</details>

---

**Q3.** 在 Kubernetes 中，「污點」（Taint）屬於哪個物件的屬性？

A) Pod
B) Node
C) DaemonSet
D) Deployment

<details>
<summary>答案</summary>
<b>B) Node</b>

污點（Taint）是 Node 節點的一個屬性，用於給節點「貼標籤」。與之相對的是 Pod 的「容忍度」（Toleration），Pod 根據容忍度來決定能否調度到帶有特定污點的節點上。
</details>

---

**Q4.** Master 節點預設帶有什麼污點效果，導致普通 Pod 無法調度到 Master 上？

A) NoExecute
B) NoSchedule
C) PreferNoSchedule
D) PreferNoExecute

<details>
<summary>答案</summary>
<b>B) NoSchedule</b>

Master 節點預設有污點 node-role.kubernetes.io/master:NoSchedule，這個污點的效果是 NoSchedule，意味著它會拒絕不能容忍該污點的 Pod 調度到本節點上運行。
</details>

---

**Q5.** 下列關於「靜態 Pod」的描述，哪一項是正確的？

A) 靜態 Pod 由 Kubernetes API Server 直接管理
B) 靜態 Pod 可以通過 kubectl delete 命令刪除
C) 靜態 Pod 的 YAML 文件預設存放在 /etc/kubernetes/manifests 目錄
D) 靜態 Pod 支持滾動更新功能

<details>
<summary>答案</summary>
<b>C) 靜態 Pod 的 YAML 文件預設存放在 /etc/kubernetes/manifests 目錄</b>

靜態 Pod 不受 Kubernetes 系統管控，不與 apiserver、scheduler 發生關係，由節點上的 kubelet 直接管理。靜態 Pod 的 YAML 文件預設存放在 /etc/kubernetes/manifests 目錄下，kubelet 會定期檢查該目錄並創建或刪除 Pod。
</details>

---

## 第二部分：是非題（每題 3 分，共 15 分）

**Q6.** DaemonSet 和 Deployment 都屬於 apps API 組。 _(對/錯)_

<details>
<summary>答案</summary>
<b>對</b> - DaemonSet 和 Deployment 都屬於 apps/v1 API 組，它們都是用於管理在線業務的控制器。
</details>

---

**Q7.** DaemonSet 可以使用 kubectl create 命令自動生成 YAML 樣板文件。 _(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - Kubernetes 不提供自動創建 DaemonSet YAML 樣板的功能，無法用 kubectl create 直接創建 DaemonSet 對象。需要通過其他方式（如手動編寫或修改 Deployment 樣板）來創建 DaemonSet 的 YAML 文件。
</details>

---

**Q8.** 如果去掉 Master 節點上的污點，DaemonSet 會自動在 Master 節點上創建 Pod。 _(對/錯)_

<details>
<summary>答案</summary>
<b>對</b> - DaemonSet 會持續監控集群節點狀態，當 Master 節點的污點被移除後，DaemonSet 會發現變化並在 Master 節點上創建一個「守護」Pod。
</details>

---

**Q9.** 「容忍度」（Toleration）只能在 DaemonSet 中使用，不能用於 Deployment 或 Job。 _(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - 容忍度是 Pod 的屬性，不是 DaemonSet 獨有的概念。可以在 Job/CronJob、Deployment 等管理 Pod 的控制器中使用 tolerations，實現更靈活的調度策略。
</details>

---

**Q10.** 靜態 Pod 與 DaemonSet 完全相同，可以互相替代使用。 _(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - 雖然靜態 Pod 可以實現與 DaemonSet 類似的效果（在每個節點上運行 Pod），但它不受 Kubernetes 控制，必須在節點上純手動部署，使用場景和管理方式都有很大差異，應當謹慎使用。
</details>

---

## 第三部分：填充題（每題 5 分，共 20 分）

**Q11.** DaemonSet 的目標是在集群的每個節點上運行且僅運行 **\_\_\_\_** 個 Pod 實例，就好像是為節點配上一隻「看門狗」。

<details>
<summary>答案</summary>
<b>一（1）</b>

DaemonSet 的核心特性是在每個符合條件的節點上運行且僅運行一個 Pod 實例，Pod 數量與節點數量保持同步。
</details>

---

**Q12.** 在 Kubernetes 中，**\_\_\_\_** 是節點的屬性，用於給節點「貼標籤」；而 **\_\_\_\_** 是 Pod 的屬性，決定 Pod 能否在帶有特定標籤的節點上運行。

<details>
<summary>答案</summary>
<b>污點（Taint）、容忍度（Toleration）</b>

污點（Taint）屬於 Node，容忍度（Toleration）屬於 Pod，兩者共同決定 Pod 的調度策略。
</details>

---

**Q13.** 如果要讓 Pod 能夠容忍 node-role.kubernetes.io/master:NoSchedule 這個污點，需要在 Pod 的 YAML 中添加 tolerations 欄位，其中 operator 通常設置為 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>Exists</b>

operator 欄位用於指定如何匹配污點，一般使用 Exists，表示存在指定名稱和效果的污點即可匹配。
</details>

---

**Q14.** Kubernetes 的核心組件（如 apiserver、etcd、scheduler、controller-manager）以 **\_\_\_\_** 的形式存在，這也是它們能夠先於 Kubernetes 集群啟動的原因。

<details>
<summary>答案</summary>
<b>靜態 Pod</b>

Kubernetes 的核心組件以靜態 Pod 的形式運行，不受 Kubernetes API Server 管理，由 kubelet 直接管理，因此可以在集群啟動前就運行。
</details>

---

## 第四部分：簡答題（每題 10 分，共 30 分）

**Q15.** 請說明 DaemonSet 和 Deployment 在使用場景上的主要區別。

<details>
<summary>答案</summary>

**主要區別：**

1. **副本數控制：**
   - DaemonSet：Pod 數量由節點數量決定，每個節點運行一個 Pod
   - Deployment：通過 replicas 手動設定副本數，可任意擴容縮容

2. **調度策略：**
   - DaemonSet：Pod 與節點綁定，確保每個節點都有守護進程
   - Deployment：Pod 可以在集群中「漂移」，不關心具體運行在哪個節點

3. **適用場景：**
   - DaemonSet：適合節點級別的守護進程，如監控代理、日誌收集、網絡代理、安全審計
   - Deployment：適常規在線業務，如 Web 應用、API 服務、資料庫等

4. **功能特性：**
   - Deployment 支持滾動更新、回滾等高級功能
   - DaemonSet 功能相對簡單，專注於節點覆蓋
</details>

---

**Q16.** 解釋「污點」和「容忍度」的工作原理，並說明它們在 Kubernetes 調度中的作用。

<details>
<summary>答案</summary>

**工作原理：**

1. **污點（Taint）：**
   - 屬於 Node 節點的屬性
   - 給節點「貼標籤」，標記節點的特殊屬性
   - 包含鍵、值和效果（Effect）三個部分
   - 常見效果：NoSchedule（不調度）、NoExecute（驅逐已運行的 Pod）

2. **容忍度（Toleration）：**
   - 屬於 Pod 的屬性
   - 定義 Pod 能夠「容忍」的污點
   - 需要指定污點的鍵、效果和匹配方式（operator）

**調度作用：**

1. **調度決策：**
   - 調度器在為 Pod 選擇節點時，會檢查節點的污點
   - 如果 Pod 沒有對應的容忍度，則不會被調度到該節點

2. **精細化控制：**
   - 可以實現「專用節點」：為特殊工作負載保留節點
   - 可以實現「隔離節點」：暫時將節點從調度池中移除
   - 可以實現「分級調度」：不同優先級的 Pod 使用不同的節點

3. **實際應用：**
   - Master 節點預設帶有污點，防止普通 Pod 調度
   - 系統組件使用容忍度在特定節點上運行
   - 結合 DaemonSet 實現節點級別的守護進程部署
</details>

---

**Q17.** 什麼是靜態 Pod？它與 DaemonSet 有什麼異同？在什麼情況下應該使用靜態 Pod？

<details>
<summary>答案</summary>

**靜態 Pod 定義：**

靜態 Pod 是一種特殊的 Pod，不受 Kubernetes API Server、Scheduler 等 Kubernetes 系統組件管控，而是由節點上的 kubelet 直接管理。

**與 DaemonSet 的異同：**

**相同點：**
- 都可以在每個節點上運行一個 Pod 實例
- 都適合運行節點級別的守護進程

**不同點：**

| 特性 | 靜態 Pod | DaemonSet |
|------|----------|-----------|
| 管理方式 | kubelet 直接管理 | Kubernetes API 管理 |
| YAML 存放位置 | 節點本地目錄 | etcd |
| 創建方式 | 手動放置文件 | kubectl apply |
| 更新方式 | 手動修改文件 | 滾動更新 |
| 可見性 | 僅在當前節點可見 | 集群範圍可見 |
| 生命周期管理 | kubelet 負責 | DaemonSet 控制器負責 |

**使用場景：**

1. **Kubernetes 核心組件：**
   - apiserver、etcd、scheduler、controller-manager
   - 需要在集群啟動前就運行的組件

2. **特殊需求：**
   - DaemonSet 無法滿足的特殊部署需求
   - 需要完全控制 Pod 的啟動時機
   - 離線環境或無法連接 API Server 的場景

3. **注意事項：**
   - 應當謹慎使用靜態 Pod
   - 缺乏 Kubernetes 的自動化管理能力
   - 維護成本較高，不適合大規模部署
</details>

---

## 第五部分：配對題（10 分）

**Q18.** 請將下列應用類型與最適合的部署方式進行配對：

| 應用類型 | 部署方式 |
|----------|----------|
| 1. Web 前端應用 | A. DaemonSet |
| 2. 節點監控代理（Prometheus Node Exporter） | B. Deployment |
| 3. 日誌收集代理（Fluentd） | C. Deployment |
| 4. 資料庫服務（MySQL） | D. DaemonSet |
| 5. 網絡代理（kube-proxy） | E. DaemonSet |

<details>
<summary>答案</summary>

| 應用類型 | 部署方式 |
|----------|----------|
| 1. Web 前端應用 | B. Deployment |
| 2. 節點監控代理（Prometheus Node Exporter） | A. DaemonSet |
| 3. 日誌收集代理（Fluentd） | D. DaemonSet |
| 4. 資料庫服務（MySQL） | C. Deployment |
| 5. 網絡代理（kube-proxy） | E. DaemonSet |

**說明：**

1. **Web 前端應用** → Deployment：需要根據流量動態擴容縮容，不依賴特定節點。

2. **節點監控代理** → DaemonSet：需要在每個節點上運行，收集節點級別的監控數據。

3. **日誌收集代理** → DaemonSet：需要在每個節點上收集容器運行時產生的日誌。

4. **資料庫服務** → Deployment：需要穩定的存儲和高可用架構，通常使用 StatefulSet 更佳，但 Deployment 也可以用於無狀態服務。

5. **網絡代理** → DaemonSet：必須每個節點都運行，否則節點無法加入 Kubernetes 網絡。
</details>

---

## 學習重點回顧

1. **DaemonSet 核心概念：**
   - 目標：在每個節點上運行一個 Pod 實例
   - 適用場景：監控、日誌、網絡代理、安全應用等守護進程

2. **YAML 配置要點：**
   - 屬於 apps/v1 API 組
   - 與 Deployment 類似，但無 replicas 欄位
   - 需要手動編寫 YAML，無法自動生成

3. **污點與容忍度：**
   - 污點（Taint）屬於 Node
   - 容忍度（Toleration）屬於 Pod
   - 共同決定 Pod 的調度策略

4. **靜態 Pod：**
   - 不受 Kubernetes 控制，由 kubelet 管理
   - YAML 存放在 /etc/kubernetes/manifests
   - 適用於集群核心組件，應謹慎使用

---

**生成來源：** Kubernetes 入門課程 - 第 19 課：DaemonSet：忠實可靠的看門狗