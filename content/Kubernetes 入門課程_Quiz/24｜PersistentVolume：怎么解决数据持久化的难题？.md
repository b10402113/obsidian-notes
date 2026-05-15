# PersistentVolume 測驗

**科目:** Kubernetes 入門課程
**難度:** 混合（基礎到進階）
**題數:** 15 題
**建議時間:** 20-25 分鐘

---

## 第一部分：選擇題 (30 分)

**Q1.** 在 Kubernetes 中，PersistentVolume (PV) 屬於什麼層級的資源？

A) Pod 層級的資源
B) Namespace 層級的資源
C) 集群層級的系統資源，與 Node 平級
D) 容器層級的資源

<details>
<summary>答案</summary>
<b>C)</b> PV 屬於集群層級的系統資源，與 Node 平級。Pod 對 PV 沒有管理權，只有使用權。一般由系統管理員維護，因為管理存儲設備已超出 Kubernetes 的能力範圍。
</details>

---

**Q2.** Kubernetes 中 PersistentVolumeClaim (PVC) 的主要作用是什麼？

A) 直接創建存儲設備
B) 代表 Pod 向系統申請存儲資源
C) 管理存儲設備的類型
D) 定義存儲設備的容量

<details>
<summary>答案</summary>
<b>B)</b> PVC 是給 Pod 使用的對象，相當於 Pod 的代理，代表 Pod 向系統申請 PV。一旦資源申請成功，Kubernetes 就會把 PV 和 PVC 關聯在一起，這個動作叫做「綁定」（bind）。
</details>

---

**Q3.** StorageClass 在 Kubernetes 存儲系統中的角色是什麼？

A) 直接創建存儲設備
B) 抽象特定類型的存儲系統，在 PVC 和 PV 之間充當協調者
C) 定義存儲容量
D) 管理 Pod 的存儲掛載

<details>
<summary>答案</summary>
<b>B)</b> StorageClass 抽象了特定類型的存儲系統（如 Ceph、NFS），在 PVC 和 PV 之間充當「協調人」的角色，幫助 PVC 找到合適的 PV，簡化 Pod 挫載「虛擬盤」的過程。
</details>

---

**Q4.** Kubernetes 中有哪三種存儲訪問模式（accessModes）？

A) ReadWrite, ReadOnly, WriteOnly
B) ReadWriteOnce, ReadOnlyOnce, ReadWriteMany
C) ReadWriteOnce, ReadOnlyMany, ReadWriteMany
D) SingleAccess, MultiAccess, SharedAccess

<details>
<summary>答案</summary>
<b>C)</b> Kubernetes 定義了三種訪問模式：
<ul>
<li>ReadWriteOnce：存儲卷可讀可寫，但只能被一個節點上的 Pod 挫載</li>
<li>ReadOnlyMany：存儲卷只讀不可寫，可以被任意節點上的 Pod 多次挫載</li>
<li>ReadWriteMany：存儲卷可讀可寫，也可以被任意節點上的 Pod 多次挫載</li>
</ul>
要注意這些訪問模式限制的對象是節點而不是 Pod，因為存儲是系統層級的概念。
</details>

---

**Q5.** 在 Kubernetes 中定義存儲容量時，應該使用什麼單位格式？

A) KB, MB, GB（基數 1024）
B) Ki, Mi, Gi（基數 1024）
C) K, M, G（基數 1000）
D) kB, mB, gB（基數 1000）

<details>
<summary>答案</summary>
<b>B)</b> Kubernetes 使用國際標準定義存儲容量，日常習慣使用的 KB/MB/GB 基數是 1024，要寫成 Ki/Mi/Gi。一定要小心不要寫錯了，否則單位不一致實際容量會對不上。
</details>

---

**Q6.** HostPath 類型的 PV 有什麼特點？

A) 可以在任意節點間遷移，支持 Pod 跨節點持久化
B) 數據存儲在節點本地，速度快但不能跟隨 Pod 遷移
C) 只能用於生產環境的關鍵應用
D) 支持多節點同時讀寫

<details>
<summary>答案</summary>
<b>B)</b> HostPath 是最簡單的一種 PV，數據存儲在節點本地，速度快但不能跟隨 Pod 遷移。如果 Pod 重建時被調度到了其他節點上，即使挫載了本地目錄，也不會是之前的存儲位置，持久化功能也就失效了。一般用來做測試或 DaemonSet 這類與節點關係密切的應用。
</details>

---

## 第二部分：是非題 (20 分)

**Q7.** Pod 可以直接在 YAML 中選擇和創建 PersistentVolume。 _(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - Pod 不能直接管理 PV。PV 是集群層級的系統資源，由系統管理員維護。Pod 需要通過 PVC 來申請存儲資源，而不是直接選擇或創建 PV。這樣的設計符合「單一職責」原則，讓 Pod 不需要關心存儲設備的專業細節。
</details>

---

**Q8.** PVC 申請的容量如果小於 PV 的容量，Kubernetes 會將整個 PV 給 PVC 使用。 _(對/錯)_

<details>
<summary>答案</summary>
<b>對</b> - 如果 PVC 申請的容量（如 5MB）小於 PV 的實際容量（如 10MB），Kubernetes 在找不到更合適的 PV 時，會將這個較大的 PV 分配給 PVC。PVC 的實際容量會是 PV 的容量（10MB），而不是最初申請的容量（5MB），多出的容量算是「福利」。
</details>

---

**Q9.** PVC 申請的容量如果大於系統中所有 PV 的容量，PVC 會自動創建新的 PV。 _(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - PVC 不會自動創建 PV。如果 PVC 申請的容量（如 100MB）大於系統中所有 PV 的容量，PVC 會一直處於 Pending 狀態，表示 Kubernetes 在系統裡沒有找到符合要求的存儲，無法分配資源。只能等待有滿足要求的 PV 才能完成綁定。
</details>

---

**Q10.** PV 的 accessModes 限制的對象是 Pod 而不是節點。 _(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - accessModes 限制的對象是節點而不是 Pod。因為存儲是系統層級的概念，不屬於 Pod 里的進程。例如 ReadWriteOnce 表示存儲卷只能被一個節點上的 Pod 挫載，而不是限制單個 Pod。
</details>

---

## 第三部分：填空題 (20 分)

**Q11.** Kubernetes 管理存儲資源的三個 API 對象分別是 **____**（簡稱 PV）、**____**（簡稱 PVC）和 **____**。

<details>
<summary>答案</summary>
<b>PersistentVolume, PersistentVolumeClaim, StorageClass</b> - PV 是對存儲設備的抽象，PVC 代表 Pod 向系統申請存儲資源，StorageClass 抽象特定類型的存儲系統，在 PVC 和 PV 之間充當協調者。
</details>

---

**Q12.** 在 PV 和 PVC 的 YAML 定義中，**____** 字段定義了存儲設備的訪問模式，目前 Kubernetes 支持的三種模式是：**____**、**____** 和 **____**。

<details>
<summary>答案</summary>
<b>accessModes, ReadWriteOnce, ReadOnlyMany, ReadWriteMany</b> - accessModes 定義了存儲設備的訪問模式，即虛擬盤的讀寫權限，類似 Linux 的文件訪問模式。
</details>

---

**Q13.** 在 Pod 的 YAML 中，要使用 PVC 挫載存儲卷，需要在 volumes 字段中使用 **____** 字段指定 PVC 的名字，然後在 containers 的 **____** 字段中將存儲卷挫載進容器。

<details>
<summary>答案</summary>
<b>persistentVolumeClaim, volumeMounts</b> - 使用 PVC 時，需要在 spec.volumes 中用 persistentVolumeClaim 字段（包含 claimName）指定 PVC 名稱，然後在 containers.volumeMounts 中指定挫載路徑。
</details>

---

**Q14.** PVC 創建成功後，Kubernetes 會根據 **____**、**____** 等條件在集群里查找符合要求的 PV，如果找到合適的存儲對象就會把它們 **____**在一起。

<details>
<summary>答案</summary>
<b>StorageClass, resources, 綁定（bind）</b> - Kubernetes 通過 StorageClass 和 resources.request 等條件查找合適的 PV，然後將 PV 和 PVC「綁定」在一起，實現存儲的分配。
</details>

---

## 第四部分：簡答題 (20 分)

**Q15.** 請說明 PV、PVC 和 StorageClass 三者之間的關係和分工，以及 Pod 如何通過這三個對象實現持久化存儲。

<details>
<summary>答案</summary>

<b>三者的關係與分工：</b>

1. <b>PersistentVolume (PV)</b>：
   - 是 Kubernetes 對存儲設備的抽象
   - 由系統管理員維護，屬於集群層級的系統資源
   - 需要描述清楚存儲設備的類型、訪問模式、容量等信息
   - 代表實際的存儲資源（如 Ceph、GlusterFS、NFS、本地磁盤等）

2. <b>PersistentVolumeClaim (PVC)</b>：
   - 代表 Pod 向系統申請存儲資源
   - 是給 Pod 使用的對象，相當於 Pod 的代理
   - 声明對存儲的「期望狀態」（類型、訪問模式、容量等）
   - 一旦申請成功，Kubernetes 會將 PV 和 PVC「綁定」

3. <b>StorageClass</b>：
   - 抽象特定類型的存儲系統（如 Ceph、NFS）
   - 在 PVC 和 PV 之間充當「協調人」角色
   - 幫助 PVC 找到合適的 PV
   - 簡化 Pod 挫載「虛擬盤」的過程

<b>Pod 現現持久化存儲的流程：</b>

1. 系統管理員創建 PV，定義存儲設備的詳細信息
2. Pod 使用者創建 PVC，声明存儲需求
3. Kubernetes 根據 PVC 的 StorageClass、accessModes、resources 等條件查找合適的 PV
4. 如果找到匹配的 PV，將 PV 和 PVC 綁定
5. Pod 在 YAML 中通過 persistentVolumeClaim 字段引用 PVC
6. Pod 啟動時，Kubernetes 將 PV 以 Volume 的形式挫載進容器
7. Pod 的數據寫入持久化存儲，即使 Pod 刪除重建，數據依然存在

<b>關鍵點：</b>
- Pod 不直接操作 PV，只通過 PVC 申請
- PV/PVC/StorageClass 使用「中間層」思想，將存儲卷的分配管理過程細化
- 這樣的設計讓 Pod 不需要關心存儲設備的專業、複雜的實現細節
</details>

---

**Q16.** 解釋為什麼 HostPath 類型的 PV 不適合用於生產環境，以及它的適用場景。

<details>
<summary>答案</summary>

<b>不適合生產環境的原因：</b>

1. <b>節點本地存儲的限制</b>：
   - HostPath 類型的 PV 將數據存儲在節點的本地目錄
   - 如果 Pod 重建時被調度到其他節點，無法訪問原節點上的存儲
   - 即使挫載了本地目錄，也不會是之前的存儲位置
   - 持久化功能失效，數據無法跨節點保持

2. <b>數據不可遷移</b>：
   - Pod 在集群中可能在任意節點上運行
   - HostPath 存儲與特定節點強绑定
   - 當 Pod 需要遷移時，數據無法跟隨 Pod 移動

3. <b>節點故障風險</b>：
   - 如果存儲所在的節點發生故障，數據可能永久丢失
   - 沒有冗余和高可用保障

<b>適用場景：</b>

1. <b>測試環境</b>：
   - 用於初步認識 PV 的用法
   - 快速驗證存儲卷的基本功能
   - 不關心數據的長期保存

2. <b>DaemonSet 類應用</b>：
   - DaemonSet 保證每個節點上都運行一個 Pod
   - Pod 與節點關係密切，不會跨節點調度
   - 可以安全使用節點本地存儲

3. <b>單節點集群</b>：
   - 只有單一節點的簡單環境
   - Pod 不會跨節點遷移
   - HostPath 的限制不會造成問題

<b>總結：</b>
HostPath 是最簡單的 PV 類型，數據存儲在節點本地，速度快但不能跟隨 Pod 遷移。適合測試或特定場景，生產環境應使用 NFS、Ceph 等支持跨節點的存儲方案。
</details>

---

## 第五部分：配對題 (10 分)

**Q17.** 將以下術語與其定義進行配對：

| 術語 | 定義 |
| --- | --- |
| 1. PersistentVolume | A. 代表 Pod 向系統申請存儲資源，声明對存儲的期望狀態 |
| 2. PersistentVolumeClaim | B. 抽象特定類型的存儲系統，協助 PVC 找到合適的 PV |
| 3. StorageClass | C. Kubernetes 對存儲設備的抽象，由系統管理員維護 |
| 4. ReadWriteOnce | D. 存儲卷可讀可寫，可被任意節點上的 Pod 多次挫載 |
| 5. ReadWriteMany | E. 存儲卷可讀可寫，但只能被一個節點上的 Pod 挫載 |

<details>
<summary>答案</summary>
1-C, 2-A, 3-B, 5-D, 4-E

<b>解釋：</b>
<ul>
<li>PersistentVolume (PV)：對存儲設備的抽象，集群層級系統資源</li>
<li>PersistentVolumeClaim (PVC)：Pod 的代理，申請存儲資源</li>
<li>StorageClass：抽象存儲類型，充當協調者</li>
<li>ReadWriteMany：多節點可讀寫</li>
<li>ReadWriteOnce：單節點可讀寫</li>
</ul>
</details>

---

**Q18.** 將以下 YAML 字段與其作用進行配對：

| YAML 字段 | 作用 |
| --- | --- |
| 1. storageClassName | A. 定義存儲卷的本地路徑（HostPath 特有） |
| 2. accessModes | B. 指定要使用的 PVC 名稱 |
| 3. capacity | C. 定義存儲類型的抽象名稱 |
| 4. hostPath | D. 定義存儲設備的容量（PV）或申請容量（PVC） |
| 5. persistentVolumeClaim | E. 定義存儲設備的訪問模式/讀寫權限 |

<details>
<summary>答案</summary>
1-C, 2-E, 3-D, 4-A, 5-B

<b>解釋：</b>
<ul>
<li>storageClassName：對存儲類型的抽象，名字可任意定義（如 host-test）</li>
<li>accessModes：定義訪問模式（ReadWriteOnce, ReadOnlyMany, ReadWriteMany）</li>
<li>capacity：在 PV 中表示實際容量，在 PVC 的 resources.request 中表示申請容量</li>
<li>hostPath：指定本地存儲路徑（如 /tmp/host-10m-pv/）</li>
<li>persistentVolumeClaim：在 Pod 的 volumes 中指定 PVC 的 claimName</li>
</ul>
</details>

---

## 答案總表

### 第一部分：選擇題
1. C
2. B
3. B
4. C
5. B
6. B

### 第二部分：是非題
7. 錯
8. 對
9. 錯
10. 錯

### 第三部分：填空題
11. PersistentVolume, PersistentVolumeClaim, StorageClass
12. accessModes, ReadWriteOnce, ReadOnlyMany, ReadWriteMany
13. persistentVolumeClaim, volumeMounts
14. StorageClass, resources, 綁定（bind）

### 第四部分：簡答題
15. 見詳細答案
16. 見詳細答案

### 第五部分：配對題
17. 1-C, 2-A, 3-B, 5-D, 4-E
18. 1-C, 2-E, 3-D, 4-A, 5-B

---

_生成來源：Kubernetes 入門課程 - 第 24 課：PersistentVolume：怎麼解決數據持久化的難題？_
_測驗類型：複合型測驗（選擇題、是非題、填空題、簡答題、配對題）_
_難度分布：基礎 40%，應用 40%，進階 20%_