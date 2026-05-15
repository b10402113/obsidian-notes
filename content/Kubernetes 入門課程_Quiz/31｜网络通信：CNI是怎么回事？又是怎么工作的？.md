# CNI 網絡通信測驗

**科目:** Kubernetes 入門課程
**難度:** 混合 (簡單 40%、中等 40%、困難 20%)
**題數:** 20 題
**建議時間:** 25 分鐘

---

## 第一部分:選擇題 (每題 5 分，共 40 分)

**Q1.** Kubernetes 的網絡模型被稱為什麼？

A) IP-per-container
B) IP-per-pod
C) Bridge-per-pod
D) NAT-per-pod

<details>
<summary>答案</summary>
<b>B) IP-per-pod</b>

Kubernetes 採用的是「IP-per-pod」網絡模型，每個 Pod 都會有唯一的一個 IP 地址，這讓 Pod 相當於一台虛擬機，而且可以直接互通，無需進行網絡地址轉換（NAT）。
</details>

---

**Q2.** 關於 Kubernetes 的網絡模型基本假設，以下哪一項是錯誤的？

A) 集群裡的每個 Pod 都會有唯一的一個 IP 地址
B) Pod 裡的所有容器共享這個 IP 地址
C) 集群裡的所有 Pod 都屬於同一個網段
D) Pod 之間通信需要進行網絡地址轉換（NAT）

<details>
<summary>答案</summary>
<b>D) Pod 之間通信需要進行網絡地址轉換（NAT）</b>

這是錯誤的說明。Kubernetes 網絡模型的第四個基本假設是「Pod 可以基於 IP 地址直接訪問另一個 Pod，<b>不需要</b>做麻煩的網絡地址轉換（NAT）」，這正是其優勢所在。
</details>

---

**Q3.** CNI 插件按照實現技術可以分成哪三種類型？

A) Bridge、Route、Direct
B) Overlay、Route、Underlay
C) VXLAN、BGP、eBPF
D) Flannel、Calico、Cilium

<details>
<summary>答案</summary>
<b>B) Overlay、Route、Underlay</b>

CNI 插件依據實現技術的不同，可以分成三種類型：
- Overlay：在底層網絡之上構建邏輯網絡，需要封包和拆包
- Route：使用系統內置的路由功能，無需封包拆包
- Underlay：直接使用底層網絡，Pod 和宿主機在同一網絡中
</details>

---

**Q4.** 關於 Overlay、Route 和 Underlay 三種 CNI 類型的特點，以下描述正確的是？

A) Overlay 對底層網絡依賴性最強，性能最高
B) Route 需要封包和拆包，性能較低
C) Underlay 性能最高，但對底層硬件依賴性最強
D) 三種類型的性能和靈活性都相同

<details>
<summary>答案</summary>
<b>C) Underlay 性能最高，但對底層硬件依賴性最強</b>

- Overlay：適應性強，但有額外傳輸成本，性能較低
- Route：性能高，但對底層網絡依賴性較強
- Underlay：對底層硬件依賴性最強，不夠靈活，但性能最高
</details>

---

**Q5.** Flannel 網絡插件默認使用的是什麼模式？

A) Route 模式，使用 BGP 協議
B) Overlay 模式，使用 VXLAN 技術
C) Underlay 模式，直接使用宿主機網絡
D) Bridge 模式，使用 docker0 網橋

<details>
<summary>答案</summary>
<b>B) Overlay 模式，使用 VXLAN 技術</b>

Flannel 默認使用基於 VXLAN 的 Overlay 模式。它最早是一種 Overlay 模式的網絡插件，後來才用 HostGateway 技術支持了 Route 模式。Flannel 簡單易用，但在性能方面表現不是太好，一般不建議在生產環境裡使用。
</details>

---

**Q6.** 關於 Flannel 的工作方式，以下描述錯誤的是？

A) Flannel 使用 cni0 網橋，而不是 docker0
B) 同一節點上的 Pod 通過 cni0 網橋直接通信
C) 跨主機通信時，數據包通過 flannel.1 設備進行 VXLAN 封裝
D) Flannel 的 Overlay 模式沒有性能損失，效率最高

<details>
<summary>答案</summary>
<b>D) Flannel 的 Overlay 模式沒有性能損失，效率最高</b>

這是錯誤的說明。Flannel 的 Overlay 模式需要對原始數據包進行封裝（封包），到達目的地後還要拆包，這個過程會有額外的傳輸成本，導致性能較低。這也是為什麼生產環境通常建議使用性能更好的插件如 Calico。
</details>

---

**Q7.** Calico 網絡插件的主要特點是什麼？

A) 使用 Overlay 模式，需要 cni0 網橋
B) 使用 Route 模式和 BGP 協議，無需網橋
C) 只支持單機環境，不支持跨主機通信
D) 必須使用 VXLAN 技術

<details>
<summary>答案</summary>
<b>B) 使用 Route 模式和 BGP 協議，無需網橋</b>

Calico 是一種 Route 模式的網絡插件，使用 BGP 協議（Border Gateway Protocol）來維護路由信息。它不使用 cni0 網橋，而是在宿主機上創建路由規則，讓數據包直接「跳」到目標網卡，因此性能比 Flannel 更高。
</details>

---

**Q8.** 關於 Cilium 網絡插件，以下哪項描述是正確的？

A) 只支持 Overlay 模式
B) 只支持 Route 模式
C) 深度使用 Linux eBPF 技術，支持多種模式
D) 是最早出現的 CNI 插件

<details>
<summary>答案</summary>
<b>C) 深度使用 Linux eBPF 技術，支持多種模式</b>

Cilium 是一個比較新的網絡插件，同時支持 Overlay 模式和 Route 模式。它的特點是深度使用了 Linux eBPF 技術，在內核層次操作網絡數據，所以性能很高，可以靈活實現各種功能。在 2021 年它加入了 CNCF，成為孵化項目。
</details>

---

## 第二部分:是非題 (每題 3 分，共 15 分)

**Q9.** Kubernetes 內置了完整的網絡實現方案，可以直接使用無需安裝網絡插件。_(是/否)_

<details>
<summary>答案</summary>
<b>否</b>

Kubernetes 只定義了網絡模型（IP-per-pod），但沒有內置實現。它制定了 CNI（Container Networking Interface）標準，需要開發者遵循這個規範來實現網絡插件，為 Pod 創建虛擬網卡、分配 IP 地址、設置路由規則等。
</details>

---

**Q10.** Docker 的 bridge 網絡模式可以輕鬆實現跨主機容器通信，無需額外配置。_(是/否)_

<details>
<summary>答案</summary>
<b>否</b>

Docker 的 bridge 網絡模式只局限在單機環境裡工作，跨主機通信非常困難，需要做端口映射和網絡地址轉換（NAT）。這正是 Kubernetes 要提出自己的網絡模型的原因之一。
</details>

---

**Q11.** 在 Flannel 網絡中，虛擬網卡對（veth pair）的特性使得 Pod 能夠連接到 cni0 網橋。_(是/否)_

<details>
<summary>答案</summary>
<b>是</b>

正確。每個 Pod 都會創建一個虛擬網卡對（veth pair），兩個虛擬網卡分別「插」在容器和網橋上。例如，Pod 內的 eth0@if45 對應宿主機上的 veth41586979@if3，而這個 veth 設備被「插」在 cni0 網橋上，這樣 Pod 就連上了網橋，可以進行通信。
</details>

---

**Q12.** Calico 使用網橋（如 cni0）來實現 Pod 之間的通信。_(是/否)_

<details>
<summary>答案</summary>
<b>否</b>

Calico 不使用 cni0 網橋。因為 Calico 是 Route 模式，它在宿主機上創建路由規則，讓數據包不經過網橋直接「跳」到目標網卡去。例如，Pod A 要訪問 Pod B，查路由表後知道要走特定的 cali 設備，數據就會直接進 Pod B 的網卡，省去了網橋的中間步驟。
</details>

---

**Q13.** CNI 通過「依賴倒置」原則將網絡實現工作交給插件，使 Kubernetes 集群擁有一個統一的網絡空間。_(是/否)_

<details>
<summary>答案</summary>
<b>是</b>

正確。CNI 為網絡插件定義了一系列通用接口，開發者只要遵循這個規範就可以接入 Kubernetes。不管下層是什麼樣的環境，不管插件是怎麼實現的，在 Kubernetes 集群裡都會有一個乾淨、整潔的網絡空間。這就是依賴倒置原則的好處。
</details>

---

## 第三部分:填充題 (每題 5 分，共 15 分)

**Q14.** Docker 創建的默認網橋名稱是 **\_\_\_\_**，而 Flannel 在 Kubernetes 中使用的網橋名稱是 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>docker0, cni0</b>

Docker 會創建一個名字叫「docker0」的網橋，默認是私有網段「172.17.0.0/16」。而 Flannel 在 Kubernetes 中使用的網橋名稱是「cni0」，從單機角度來看，Flannel 的網絡結構和 Docker 幾乎一樣，只是網橋名稱不同。
</details>

---

**Q15.** 在 Flannel 的跨主機通信中，假設 master 節點的 Pod 要訪問 worker 節點的 Pod，數據包會經過 **\_\_\_\_** 設備進行 VXLAN 封裝，然後通過宿主機的物理網卡發送到目標節點。

<details>
<summary>答案</summary>
<b>flannel.1</b>

根據路由表，凡是目標網段「10.10.1.0/24」的數據都要讓 flannel.1 來處理，這樣就進入了 Flannel 插件的工作流程。Flannel 會在原始網絡包前面加上額外信息，封裝成 VXLAN 報文，用宿主機網卡發出去。
</details>

---

**Q16.** Calico 使用 **\_\_\_\_** 協議（Border Gateway Protocol）來維護路由信息，這使得它的性能比 Flannel 更好，而且支持多種網絡策略。

<details>
<summary>答案</summary>
<b>BGP</b>

Calico 是一種 Route 模式的網絡插件，使用 BGP 協議（Border Gateway Protocol）來維護路由信息。BGP 是一種路由協議，能夠讓 Calico 高效地管理集群中的路由規則，實現 Pod 的跨主機通信。
</details>

---

## 第四部分:簡答題 (每題 10 分，共 20 分)

**Q17.** 請簡要說明 Flannel 的 Overlay 模式和 Calico 的 Route 模式在實現跨主機 Pod 通信時的主要區別。

<details>
<summary>答案</summary>

**Flannel 的 Overlay 模式：**
1. 使用 VXLAN 技術對數據包進行封裝（封包）
2. 數據包經過 cni0 網橋，然後通過 flannel.1 設備進行封裝
3. 封裝後的 VXLAN 報文通過宿主機網卡發送到目標節點
4. 目標節點收到後進行拆包，再交給目標 Pod
5. 有封包和拆包的開銷，性能較低

**Calico 的 Route 模式：**
1. 不使用網橋，而是依賴路由表
2. 直接在宿主機上創建路由規則
3. 數據包根據路由表直接「跳」到目標網卡
4. 無需封包和拆包過程
5. 性能更高，但對底層網絡依賴性較強

**主要區別總結：**
- Flannel 需要封包/拆包，Calico 直接路由
- Flannel 使用網橋，Calico 不使用網橋
- Calico 性能更高，但對網絡環境要求更嚴格
</details>

---

**Q18.** 請列舉 Kubernetes 網絡模型（IP-per-pod）的四個基本假設，並說明這個模型的優勢。

<details>
<summary>答案</summary>

**四個基本假設：**

1. **每個 Pod 都有唯一 IP 地址** - 集群裡的每個 Pod 都會有唯一的一個 IP 地址
2. **容器共享 IP** - Pod 裡的所有容器共享這個 IP 地址
3. **統一網段** - 集群裡的所有 Pod 都屬於同一個網段
4. **直接通信** - Pod 可以基於 IP 地址直接訪問另一個 Pod，不需要做網絡地址轉換（NAT）

**模型優勢：**

1. **簡單易管理** - Pod 擺脫了主機的硬限制，是一個「平坦」的網絡模型
2. **通信自然** - 所有 Pod 都有獨立 IP，可以直接互通，通信方式簡單
3. **易於實施其他功能** - 可以很容易地實施域名解析、負載均衡、服務發現等工作
4. **兼容現有經驗** - 以前的運維經驗都能夠直接使用
5. **友好遷移** - 對應用的管理和遷移都非常友好，每個 Pod 相當於一台虛擬機
</details>

---

## 第五部分:配對題 (共 10 分)

**Q19.** 請將以下 CNI 插件與其特性進行配對：

| CNI 插件 | 特性描述 |
| -------- | -------- |
| 1. Flannel | A. 深度使用 Linux eBPF 技術，同時支持 Overlay 和 Route 模式，2021 年加入 CNCF |
| 2. Calico | B. 最早出現的 CNI 插件之一，簡單易用，默認使用 VXLAN Overlay 模式，性能較低 |
| 3. Cilium | C. 使用 BGP 協議的 Route 模式插件，性能好，支持網絡策略、數據加密等功能 |

<details>
<summary>答案</summary>
<b>1-B, 2-C, 3-A</b>

**配對說明：**

1. **Flannel - B**：由 CoreOS 開發，是最早的 CNI 插件之一。默認使用 VXLAN Overlay 模式，簡單易用，但因為需要封包拆包，性能較低，一般不建議在生產環境使用。

2. **Calico - C**：Route 模式插件，使用 BGP 協議維護路由信息。不使用網橋，直接通過路由表讓數據包跳到目標網卡，性能比 Flannel 好，還支持網絡策略、數據加密、安全隔離、流量整形等功能。

3. **Cilium - A**：較新的網絡插件，深度使用 Linux eBPF 技術在內核層次操作網絡數據。同時支持 Overlay 和 Route 模式，性能很高，2021 年加入 CNCF 成為孵化項目，非常有前途。
</details>

---

## 總結

本測驗涵蓋了 Kubernetes 網絡通信的核心概念，包括：

- **Kubernetes 網絡模型**：IP-per-pod 的四個基本假設及其優勢
- **CNI 標準**：三種實現類型（Overlay、Route、Underlay）及其特點
- **主流插件**：Flannel、Calico、Cilium 的工作原理和比較
- **技術細節**：veth pair、網橋、路由表、VXLAN、BGP 等核心概念

掌握這些知識對於理解 Kubernetes 集群的網絡通信至關重要，也有助於在實際場景中選擇合適的網絡插件。

---

*生成自: 31｜网络通信：CNI是怎么回事？又是怎么工作的？*