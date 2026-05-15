# Kubernetes 系統監控測驗

**主題:** Metrics Server 和 Prometheus 監控系統
**難度:** 混合 (基礎 60%, 中級 30%, 進階 10%)
**題數:** 15
**建議時間:** 30 分鐘

---

## 第一部分:選擇題 (每題 4 分)

**Q1.** 在 Kubernetes 中,下列哪個命令可以查看節點和 Pod 的資源使用率?

A) kubectl status
B) kubectl top
C) kubectl monitor
D) kubectl metrics

<details>
<summary>答案</summary>
<b>B)</b> kubectl top 是 Kubernetes 提供的資源監控命令,類似於 Linux 的 top 命令。它需要安裝 Metrics Server 插件才能正常運作,包含 node 和 pod 兩個子命令,分別查看節點和 Pod 的資源狀況。
</details>

---

**Q2.** Metrics Server 收集資源指標時,從哪個組件獲取數據?

A) API Server
B) etcd
C) kubelet
D) kube-proxy

<details>
<summary>答案</summary>
<b>C)</b> Metrics Server 定時從所有節點的 kubelet 採集節點和 Pod 的指標信息,然後將這些信息交給 API Server,供 kubectl top 和 HPA 使用。
</details>

---

**Q3.** 部署 Metrics Server 時,添加 --kubelet-insecure-tls 參數的主要目的是什麼?

A) 提高性能
B) 簡化實驗環境部署,跳過 TLS 驗證
C) 啟用加密通信
D) 減少記憶體佔用

<details>
<summary>答案</summary>
<b>B)</b> Metrics Server 默認使用 TLS 協議驗證證書才能與 kubelet 安全通信,但在實驗環境中沒有必要,添加此參數可以跳過 TLS 验證,使部署工作更簡單。生產環境中應慎用此參數。
</details>

---

**Q4.** HorizontalPodAutoscaler (HPA) 不能應用於哪種控制器?

A) Deployment
B) StatefulSet
C) DaemonSet
D) ReplicaSet

<details>
<summary>答案</summary>
<b>C)</b> HPA 專門用於自動伸缩 Pod 數量,適用於 Deployment 和 StatefulSet,但不能用於 DaemonSet,因為 DaemonSet 在每個節點上只運行一個 Pod,其 Pod 数量由節點數决定,無法通過 HPA 调整。
</details>

---

**Q5.** Prometheus 的核心數據存儲組件是什麼?

A) MySQL
B) Redis
C) TSDB (Time Series Database)
D) MongoDB

<details>
<summary>答案</summary>
<b>C)</b> Prometheus Server 內部包含一個時序數據庫 TSDB,專門用來存儲監控數據。這種數據庫設計專為時間序列數據優化,能高效存儲和查詢按時間順序排列的指標數據。
</details>

---

## 第二部分:是非題 (每題 2 分)

**Q6.** Metrics Server 每個節點只佔用約 1m CPU 和 2MB 記憶體,對集群性能影響極小。_(True/False)_

<details>
<summary>答案</summary>
<b>True</b> - Metrics Server 設計輕量高效,每個節點僅佔用約 1m 的 CPU 和 2MB 的記憶體,資源消耗極小,因此性價比非常高,適合在生產環境中使用。
</details>

---

**Q7.** Prometheus 使用 Push 模式主動推送數據到各個目標。_(True/False)_

<details>
<summary>答案</summary>
<b>False</b> - Prometheus 默認使用 Pull 模式,由 Retrieval 組件定期從各個目標拉取數據。只有通過 Push Gateway 才能將 Pull 模式轉變為 Push 模式,以適配特殊的監控目標。
</details>

---

**Q8.** 使用 kubectl autoscale 命令創建 HPA 时,必須先確保 Pod 已定義 resources 資源配額。_(True/False)_

<details>
<summary>答案</summary>
<b>True</b> - HPA 完全依賴 Metrics Server 獲取運行指標,如果 Pod 沒有在 spec 中用 resources 字段定義資源配額(requests 和 limits),Metrics Server 就無法獲取 Pod 的指標,HPA 也無法實現自動化擴缩容。
</details>

---

**Q9.** Grafana 是 Prometheus 的核心組件之一。_(True/False)_

<details>
<summary>答案</summary>
<b>False</b> - Grafana 是一個獨立的開源項目,並非 Prometheus 的核心組件。它是一個通用的可视化平台,可以與多種數據源集成(包括 Prometheus),提供強大的圖形化界面和監控儀表盤功能。
</details>

---

**Q10.** HPA 的自動擴縮容是立即執行的,不需要等待 Metrics Server 收集數據。_(True/False)_

<details>
<summary>答案</summary>
<b>False</b> - Metrics Server 大約每 15 秒採集一次數據,HPA 的自動擴容和缩容也是按照這個時間點逐步處理的。當 CPU 使用率超過阈值時,HPA 會以 2 的倍數開始扩容,持續監控一段時間後才會缩容。
</details>

---

## 第三部分:填充題 (每題 4 分)

**Q11.** Metrics Server 部署後,可以使用命令 **\_\_\_\_\_\_\_\_\_\_** 查看節點資源使用率,使用 **\_\_\_\_\_\_\_\_\_\_** 查看 Pod 資源使用率。

<details>
<summary>答案</summary>
<b>kubectl top node, kubectl top pod</b>
</details>

---

**Q12.** Prometheus 在 CNCF 中的地位非常重要,它是繼 Kubernetes 之後第二個加入 CNCF 的項目,並在 **\_\_\_\_\_\_\_\_\_\_** 年順利畢業,成為云原生監控領域的 **\_\_\_\_\_\_\_\_\_\_**。

<details>
<summary>答案</summary>
<b>2018, "事實標準"</b>
</details>

---

**Q13.** 在 Prometheus 架構中,**\_\_\_\_\_\_\_\_\_\_** 用於适配特殊監控目標,將 Pull 模式轉為 Push 模式;**\_\_\_\_\_\_\_\_\_\_** 是告警中心,預設規則後通過郵件等方式告警。

<details>
<summary>答案</summary>
<b>Push Gateway, Alert Manager</b>
</details>

---

## 第四部分:簡答題 (每題 8 分)

**Q14.** 請說明 HorizontalPodAutoscaler (HPA) 的三個核心參數及其作用,並描述 HPA 的自動擴縮容工作流程。

<details>
<summary>答案</summary>
<b>三個核心參數:</b>
- min: Pod 数量的最小值,也就是缩容的下限
- max: Pod 数量的最大值,也就是扩容的上限
- cpu-percent: CPU 使用率指标,當大於此值時扩容,小於此值時缩容

<b>工作流程:</b>
1. HPA 從 Metrics Server 獲取應用的 CPU 使用率指標
2. Metrics Server 每 15 秒採集一次數據
3. 當 CPU 使用率超過預設阈值時,HPA 以 2 的倍數開始扩容,直到數量上限
4. 持續監控一段時間,如果 CPU 使用率回落,再缩容到最小值

<b>關鍵要點:</b>
- Pod 必定義 resources 資源配額才能被 HPA 監控
- 扩缩容是逐步處理的,非立即執行
</details>

---

**Q15.** 請比較 Metrics Server 和 Prometheus 的主要差異,包括功能範圍、數據類型、使用場景和部署方式。

<details>
<summary>答案</summary>
<b>主要差異比較:</b>

<b>1. 功能範圍:</b>
- Metrics Server: 僅收集核心資源指標(CPU、記憶體),輕量級插件
- Prometheus: 全面監控系統,可收集多種應用運行狀況指標

<b>2. 數據類型:</b>
- Metrics Server: 只有 CPU 和記憶體使用率
- Prometheus: 支持多種指標(内存、網絡、磁盤、應用自定義指標等)

<b>3. 使用場景:</b>
- Metrics Server: 支持 kubectl top 命令,為 HPA 提供指標數據
- Prometheus: 生产環境全面監控,配合 Grafana 提供可视化儀表盤

<b>4. 部署方式:</b>
- Metrics Server: 單個 YAML 文件,部署簡單,佔用 kube-system 名字空間
- Prometheus: 組件众多,需使用 kube-prometheus 項目,部署複雜,佔用 monitoring 名字空間

<b>5. 數據存儲:</b>
- Metrics Server: 不持久化存儲,僅提供當前狀態
- Prometheus: 使用 TSDB 時序數據庫持久化存儲歷史數據

<b>關鍵要點:</b>
- Metrics Server 是基礎必需,轻量高效
- Prometheus 是進階選項,功能全面
- 二者可配合使用,互補優勢
</details>

---

## 第五部分:配對題 (每題 6 分)

**Q16.** 將下列 Prometheus 組件與其功能進行配對:

| 組件                  | 功能描述                        |
| --------------------- | ------------------------------- |
| 1. Prometheus Server  | A. 告警中心,預設規則發送告警通知 |
| 2. Push Gateway       | B. 時序數據庫存儲監控數據       |
| 3. Alert Manager      | C. 將 Pull 模式轉為 Push 模式   |
| 4. Grafana            | D. 图形化界面和監控儀表盤       |

<details>
<summary>答案</summary>
<b>1-B, 2-C, 3-A, 4-D</b>

<b>說明:</b>
- Prometheus Server 包含 TSDB(時序數據库)和 Retrieval(數據採集)组件
- Push Gateway 用於适配無法被 Pull 的監控目標
- Alert Manager 根據預設規則觸發告警,支持郵件等多種通知方式
- Grafana 提供可視化界面,內置大量監控儀表盤模板
</details>

---

**Q17.** 將下列 Kubernetes 监控命令與其用途進行配對:

| 命令                   | 用途                         |
| ---------------------- | ---------------------------- |
| 1. kubectl top node    | A. 查看 Pod 資源使用率       |
| 2. kubectl top pod     | B. 创建 HorizontalPodAutoscaler |
| 3. kubectl autoscale   | C. 查看節點資源使用率        |
| 4. kubectl get hpa     | D. 查看 HPA 狀態             |

<details>
<summary>答案</summary>
<b>1-C, 2-A, 3-B, 4-D</b>

<b>說明:</b>
- kubectl top node 查看所有節點的 CPU 和記憶體使用率
- kubectl top pod 查看指定名字空間內 Pod 的資源使用率
- kubectl autoscale 命令可创建 HPA,需指定 min、max、cpu-percent 參數
- kubectl get hpa 可查看 HPA 的當前狀態和 Pod 數量變化
</details>

---

## 總結

本測驗涵蓋 Kubernetes 系統監控的核心概念:

1. Metrics Server: 輕量級資源指標收集工具,支持 kubectl top 命令和 HPA 功能
2. HorizontalPodAutoscaler: 基於 CPU 使用率自動伸缩 Pod 数量,實現應用自動化扩縮容
3. Prometheus: 全面監控系統,包含 TSDB、PromQL、Alert Manager、Grafana 等组件

**學習重點:**
- Metrics Server 的部署要点(--kubelet-insecure-tls 參數、镜像下载)
- HPA 的三個核心參數和工作流程
- Prometheus 架構和各组件功能
- 監控數據的採集方式(Pull vs Push)
- 實際部署過程中的準備工作

---

*資料来源: Kubernetes 入門課程 - 第 30 講「系统监控:如何使用 Metrics Server 和 Prometheus？」*