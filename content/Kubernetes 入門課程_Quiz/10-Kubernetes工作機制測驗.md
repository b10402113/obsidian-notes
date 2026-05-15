# Kubernetes 工作機制測驗

**課程：** Kubernetes 入門課程
**主題：** 自動化的運維管理：探究Kubernetes工作機制的奧秘
**難度：** 中等
**題數：** 25題
**建議時間：** 30分鐘

---

## 第一部分：選擇題 (每題4分，共40分)

**Q1.** Kubernetes 被稱為「雲時代的操作系統」，主要原因是什么？

A) 它可以直接管理硬件資源如 CPU 和內存
B) 它能夠在集群級別管理應用和服務器，實現資源管理和作業調度
C) 它只能在單機上運行，類似於 Linux
D) 它取代了傳統操作系統的所有功能

<details>
<summary>答案</summary>
<b>B)</b> Kubernetes 可以在集群級別管理應用和服務器，實現資源管理和作業調度。它不是運行在單機上管理單台計算資源，而是運行在多台服務器上管理大規模的計算資源和進程。A 錯誤，因為 Kubernetes 不直接管理硬件，而是管理抽象後的資源；C 錯誤，它管理的是集群而非單機；D 錯誤，它補充而非取代傳統操作系統。
</details>

---

**Q2.** 在 Kubernetes 的「控制面/數據面」架構中，以下哪個組件負責容器的編排工作？

A) apiserver
B) etcd
C) scheduler
D) controller-manager

<details>
<summary>答案</summary>
<b>C)</b> scheduler 負責容器的編排工作，檢查節點的資源狀態，把 Pod 調度到最適合的節點上運行，相當於部署人員。A 錯誤，apiserver 是系統的唯一入口；B 錯誤，etcd 是分布式數據庫；D 錯誤，controller-manager 負責維護資源狀態。
</details>

---

**Q3.** 關於 Kubernetes 中 Master 和 Node 的關係，以下哪項描述是正確的？

A) Master 和 Node 的劃分是絕對的，不能互相轉換
B) 在小規模集群中，Master 可以承擔 Node 的工作
C) Master 只能有一個，Node 可以有多個
D) Node 不需要 Master 就能獨立工作

<details>
<summary>答案</summary>
<b>B)</b> Master 和 Node 的劃分不是絕對的。當集群規模較小、工作負載較少時，Master 也可以承擔 Node 的工作，如 minikube 環境就只有一個節點，既作為 Master 又作為 Node。A 錯誤，劃分不是絕對的；C 錯誤，Master 可以有多個；D 錯誤，Node 需要 Master 進行調度和管理。
</details>

---

**Q4.** etcd 在 Kubernetes 中的作用是什麼？

A) 負責容器的啟動和停止
B) 管理集群的網絡通信
C) 持久化存儲系統的資源對象和狀態
D) 調度 Pod 到節點上

<details>
<summary>答案</summary>
<b>C)</b> etcd 是一個高可用的分布式 Key-Value 數據庫，用來持久化存儲系統里的各種資源對象和狀態，相當於 Kubernetes 的配置管理員。A 錯誤，這是 kubelet 和 container-runtime 的職責；B 錯誤，這是 kube-proxy 的職責；D 錯誤，這是 scheduler 的職責。
</details>

---

**Q5.** 以下哪個組件是 Kubernetes Master 中唯一的入口？

A) etcd
B) scheduler
C) apiserver
D) controller-manager

<details>
<summary>答案</summary>
<b>C)</b> apiserver 是 Master 節點同時也是整個 Kubernetes 系統的唯一入口，它對外公開了一系列的 RESTful API，並且加上了驗證、授權等功能，所有其他組件都只能和它直接通信。其他選項都不是系統入口。
</details>

---

**Q6.** kubelet 在 Kubernetes 節點中的作用是什麼？

A) 負責網絡代理和負載均衡
B) 管理容器和鏡像的實際運行
C) 作為 Node 的代理，負責管理節點相關的大部分操作
D) 存儲集群的配置信息

<details>
<summary>答案</summary>
<b>C)</b> kubelet 是 Node 的代理，負責管理 Node 相關的絕大部分操作，Node 上只有它能夠與 apiserver 通信，實現狀態報告、命令下發、啟停容器等功能，相當於 Node 上的一個「小管家」。A 錯誤，這是 kube-proxy 的職責；B 錯誤，這是 container-runtime 的職責；D 錯誤，這是 etcd 的職責。
</details>

---

**Q7.** 在 Kubernetes 中，哪個組件負責實現 TCP/UDP 反向代理？

A) kubelet
B) kube-proxy
C) container-runtime
D) scheduler

<details>
<summary>答案</summary>
<b>B)</b> kube-proxy 是 Node 的網絡代理，只負責管理容器的網絡通信，簡單來說就是為 Pod 轉發 TCP/UDP 數據包，相當於專職的「小郵差」。其他選項都不負責網絡代理功能。
</details>

---

**Q8.** 關於 Kubernetes 的 DevOps 角色，以下哪項描述是正確的？

A) 開發人員和運維人員分工明確，不能越界
B) 開發和運維的界限變得模糊，需要早期介入彼此的工作
C) 只需要開發人員，不需要運維人員
D) 運維人員負責所有 Kubernetes 的管理工作

<details>
<summary>答案</summary>
<b>B)</b> 在 Kubernetes 里，開發和運維的界限變得不那么清晰。由於雲原生的興起，開發人員從一開始就必須考慮後續的部署運維工作，而運維人員也需要在早期介入開發，才能做好應用的運維監控工作。A 是傳統模式；C 和 D 都不符合 Kubernetes 的實際工作模式。
</details>

---

**Q9.** 以下哪個不是 Kubernetes Node 的核心組件？

A) kubelet
B) kube-proxy
C) container-runtime
D) scheduler

<details>
<summary>答案</summary>
<b>D)</b> scheduler 是 Master 的組件，負責容器的編排和調度工作，而不是 Node 的組件。Node 有 3 個核心組件：kubelet、kube-proxy 和 container-runtime。
</details>

---

**Q10.** Kubernetes 中必備的插件有哪些？

A) Dashboard 和 Ingress
B) DNS 和 Dashboard
C) DNS 和 Prometheus
D) Dashboard 和 Grafana

<details>
<summary>答案</summary>
<b>B)</b> 比較重要的插件有兩個：DNS 和 Dashboard。DNS 在 Kubernetes 集群里實現了域名解析服務，能夠讓我們以域名而不是 IP 地址的方式來互相通信，是服務發現和負載均衡的基礎。Dashboard 為 Kubernetes 提供了圖形化的操作界面。
</details>

---

## 第二部分：判斷題 (每題3分，共15分)

**Q11.** Kubernetes 限定 container-runtime 必須是 Docker，不能使用其他容器運行時。 _(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - Kubernetes 的定位是容器編排平台，所以它沒有限定 container-runtime 必須是 Docker，完全可以替換成任何符合標準的其他容器運行時，例如 containerd、CRI-O 等等。
</details>

---

**Q12.** kubelet 被容器化運行在集群的 Pod 里，可以通過 docker ps 命令查看。 _(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - kubelet 因為必須要管理整個節點，容器化會限制它的能力，所以它必須在 container-runtime 之外運行。用 docker ps 是找不到 kubelet 的，需要用操作系統的 ps 命令來查看。
</details>

---

**Q13.** 在 Kubernetes 中，etcd 只與 apiserver 有直接聯系，其他組件想要讀寫 etcd 里的數據都必須經過 apiserver。 _(對/錯)_

<details>
<summary>答案</summary>
<b>對</b> - etcd 是 Kubernetes 的配置管理員，只與 apiserver 有直接聯系，任何其他組件想要讀寫 etcd 里的數據都必須經過 apiserver，這樣可以確保數據訪問的安全性和一致性。
</details>

---

**Q14.** controller-manager 負責調度 Pod 到最適合的節點上運行。 _(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - scheduler 負責調度 Pod 到最適合的節點上運行。controller-manager 的職責是維護容器和節點等資源的狀態，實現故障檢測、服務遷移、應用伸縮等功能，相當於監控運維人員。
</details>

---

**Q15.** Kubernetes 的插件（Addon）是必須安裝的，否則集群無法正常運行。 _(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - 插件是 Kubernetes 的一些附加功能，屬於「錦上添花」，不安裝也不會影響 Kubernetes 的正常運行。組件實現了 Kubernetes 的核心功能特性，沒有這些組件 Kubernetes 才無法啟動。
</details>

---

## 第三部分：填空題 (每題4分，共20分)

**Q16.** Kubernetes 采用「控制面/數據面」架構，控制面的節點叫做 **\_\_\_\_**，數據面的節點叫做 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>Master Node, Worker Node</b> - 控制面的節點叫做 Master Node，一般簡稱為 Master，它是整個集群最重要的部分。數據面的節點叫做 Worker Node，一般簡稱為 Worker 或者 Node，在 Master 的指揮下干活。
</details>

---

**Q17.** Kubernetes 的核心組件可分為兩類：實現核心功能的 **\_\_\_\_** 和增強管理能力的 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>組件（Component）, 插件（Addon）</b> - 組件實現了 Kubernetes 的核心功能特性，沒有這些組件 Kubernetes 就無法啟動，而插件則是 Kubernetes 的一些附加功能，屬於「錦上添花」，不安裝也不會影響 Kubernetes 的正常運行。
</details>

---

**Q18.** Master 里的四個組件分別是：**\_\_\_\_**、**\_\_\_\_**、**\_\_\_\_** 和 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>apiserver, etcd, scheduler, controller-manager</b> - 這四個構成了 Kubernetes 的控制平面：apiserver 是系統唯一入口；etcd 是分布式數據庫；scheduler 負責調度；controller-manager 負責狀態維護。
</details>

---

**Q19.** Node 里的三個組件分別是：**\_\_\_\_**、**\_\_\_\_** 和 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>kubelet, kube-proxy, container-runtime</b> - kubelet 是 Node 的代理；kube-proxy 負責網絡代理；container-runtime 是容器和鏡像的實際使用者。
</details>

---

**Q20.** 在 Kubernetes 中，DNS 插件實現了 **\_\_\_\_** 服務，能夠讓我們以 **\_\_\_\_** 而不是 IP 地址的方式來互相通信。

<details>
<summary>答案</summary>
<b>域名解析, 域名</b> - DNS 在 Kubernetes 集群里實現了域名解析服務，能夠讓我們以域名而不是 IP 地址的方式來互相通信，是服務發現和負載均衡的基礎。
</details>

---

## 第四部分：簡答題 (每題10分，共20分)

**Q21.** 請簡述 Kubernetes 的工作流程，說明 Master 和 Node 組件是如何協作的。

<details>
<summary>答案</summary>
Kubernetes 的工作流程如下：

1. **狀態報告**：每個 Node 上的 kubelet 會定期向 apiserver 上報節點狀態，apiserver 再存到 etcd 里。

2. **網絡服務**：每個 Node 上的 kube-proxy 實現了 TCP/UDP 反向代理，讓容器對外提供穩定的服務。

3. **Pod 調度**：scheduler 通過 apiserver 得到當前的節點狀態，調度 Pod，然後 apiserver 下發命令給某個 Node 的 kubelet，kubelet 調用 container-runtime 啟動容器。

4. **狀態監控**：controller-manager 也通過 apiserver 得到實時的節點狀態，監控可能的異常情況，再使用相應的手段去調節恢復。

<b>關鍵點：</b>
- apiserver 是所有通信的中樞
- etcd 存儲所有狀態信息
- 各組件協作實現自動化運維
</details>

---

**Q22.** 為什麼說 Kubernetes 是「雲時代的操作系統」？它與傳統操作系統有什么異同？

<details>
<summary>答案</summary>
<b>Kubernetes 作為操作系統的特點：</b>

1. **資源管理**：既可以管理軟件（應用、進程），也可以管理硬件（CPU、內存、硬盤、網卡）

2. **抽象功能**：從繁瑣的底層事務中抽象出簡潔的概念，基於這些概念去管理系統資源

<b>與傳統操作系統的相同點：</b>
- 都提供資源管理和作業調度功能
- 都有抽象概念來簡化管理

<b>與傳統操作系統的不同點：</b>

1. **管理規模**：
   - 傳統 OS：運行在單機上管理單台計算資源
   - Kubernetes：運行在多台服務器上管理幾百幾千台計算資源

2. **用戶角色**：
   - 傳統 OS：Dev 和 Ops 兩類人，分工明確
   - Kubernetes：只有 DevOps 一類人，開發和運維界限模糊

3. **抽象層次**：
   - 傳統 OS：直接管理硬件資源
   - Kubernetes：管理抽象後的池化資源，規模更大

<b>關鍵點：</b>Kubernetes 把原先繁瑣低效的人力工作搬進了高效的計算機里，能夠隨時發現集群里的變化和異常，自動維護集群的健康狀態。
</details>

---

## 第五部分：配對題 (共5分)

**Q23.** 請將以下 Kubernetes 組件與其對應的角色進行配對：

| 組件 | 角色 |
| --- | --- |
| 1. apiserver | A. 監控運維人員 |
| 2. etcd | B. 部署人員 |
| 3. scheduler | C. 聯絡員 |
| 4. controller-manager | D. 配置管理員 |

<details>
<summary>答案</summary>
1-C, 2-D, 3-B, 4-A

<b>解析：</b>
- apiserver：整個系統的唯一入口，所有組件都只能和它通信，相當於聯絡員
- etcd：持久化存儲資源對象和狀態，相當於配置管理員
- scheduler：檢查節點資源狀態，調度 Pod，相當於部署人員
- controller-manager：維護資源狀態，實現故障檢測等功能，相當於監控運維人員
</details>

---

**Q24.** 請將以下 Kubernetes Node 組件與其對應的功能進行配對：

| 組件 | 功能 |
| --- | --- |
| 1. kubelet | A. 轉發 TCP/UDP 數據包 |
| 2. kube-proxy | B. 真正干活的「苦力」 |
| 3. container-runtime | C. Node 上的「小管家」 |

<details>
<summary>答案</summary>
1-C, 2-A, 3-B

<b>解析：</b>
- kubelet：管理 Node 相關的大部分操作，與 apiserver 通信，相當於「小管家」
- kube-proxy：管理容器的網絡通信，轉發 TCP/UDP 數據包，相當於「小郵差」
- container-runtime：容器和鏡像的實際使用者，創建容器管理 Pod 生命周期，是真正干活的「苦力」
</details>

---

**Q25.** 請將以下 Kubernetes 術語與其對應的描述進行配對：

| 術語 | 描述 |
| --- | --- |
| 1. Master Node | A. 資源池，在池里分配資源，調度應用 |
| 2. Worker Node | B. 集群的「大腦和心臟」，執行管理維護工作 |
| 3. Node 池 | C. 集群的「手和腳」，在指揮下干活 |

<details>
<summary>答案</summary>
1-B, 2-C, 3-A

<b>解析：</b>
- Master Node：整個集群最重要的部分，執行集群的管理維護工作，是「大腦和心臟」
- Worker Node：在 Master 的指揮下干活，是集群的「手和腳」
- Node 池：構成資源池，Kubernetes 在這個池里分配資源，調度應用
</details>

---

## 答題說明

- **選擇題**：選擇最佳答案，只有一個正確選項
- **判斷題**：判斷陳述是否正確
- **填空題**：填寫一個或多個關鍵詞
- **簡答題**：用簡潔的語言回答問題，需包含關鍵要點
- **配對題**：將左側項目與右側正確描述配對

---

_Generated from: 10｜自动化的运维管理：探究Kubernetes工作机制的奥秘.pdf_
_Course: Kubernetes 入門課程_