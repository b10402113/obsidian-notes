# Kubernetes「棄用 Docker」是怎麼回事？測驗

**科目:** Kubernetes 入門課程
**難度:** 中等
**題數:** 20題
**建議時間:** 25分鍾

---

## 第一部分：選擇題 (共8題，每題2分)

**Q1.** Kubernetes 在哪個版本首次引入了 CRI (Container Runtime Interface)？

A) Kubernetes 1.0 (2015年)
B) Kubernetes 1.5 (2016年)
C) Kubernetes 1.10 (2018年)
D) Kubernetes 1.20 (2020年)

<details>
<summary>答案</summary>
<b>B) Kubernetes 1.5 (2016年)</b>

CRI 是在 2016 年底的 Kubernetes 1.5 版本中引入的。這是一套全新的接口標準,採用 ProtoBuffer 和 gRPC,規定 kubelet 如何調用容器運行時去管理容器和鏡像。這標誌著 Kubernetes 開始與 Docker 解耦的過程。

其他選項錯誤原因：
- A: 1.0 版本在 2015年發布,當時 Kubernetes 選擇直接依賴 Docker
- C: 1.10 版本在 2018年發布,此時 containerd 已與 Kubernetes 集成
- D: 1.20 版本在 2020年發布,是 Kubernetes 正式宣布棄用 Docker 支持的版本
</details>

---

**Q2.** 「dockershim」的主要作用是什麼？

A) 替代 Docker 成為新的容器運行時
B) 作為 kubelet 和 Docker 之間的適配器,將 Docker 接口轉換成 CRI 標準接口
C) 提供容器鏡像的存儲和管理功能
D) 增強 Docker 的性能和安全性

<details>
<summary>答案</summary>
<b>B) 作為 kubelet 和 Docker 之間的適配器,將 Docker 接口轉換成 CRI 準接口</b>

dockershim 是一個「適配器」或「墊片」(shim),夾在 kubelet 和 Docker 之間,把 Docker 的接口轉換成符合 CRI 標準的接口。這使得 Kubernetes 能夠在保持使用 Docker 的同時,具備與 Docker 解耦的條件。

其他選項錯誤原因：
- A: dockershim 不是運行時,只是接口轉換器
- C: 鏡像管理由 containerd 或 Docker Engine 處理
- D: dockershim 不提供性能或安全增強功能
</details>

---

**Q3.** containerd 最初是如何產生的？

A) Google 為 Kubernetes 開發的全新容器運行時
B) Docker 公司將 Docker Engine 重構後捐獻給 CNCF 的部分
C) CNCF 獨立開發的容器管理工具
D) Kubernetes 社區為替代 Docker 創建的項目

<details>
<summary>答案</summary>
<b>B) Docker 公司將 Docker Engine 重構後捐獻給 CNCF 的部分</b>

面對 Kubernetes 的壓力,Docker 采取「斷臂求生」策略,推動自身重構,把原本單體架構的 Docker Engine 拆分成多個模塊,其中的 Docker daemon 部分捐獻給了 CNCF,形成了 containerd。

其他選項錯誤原因：
- A: containerd 源自 Docker,不是 Google 開發
- C: 不是 CNCF 狀立開發,而是 Docker 捐獻
- D: 不是 Kubernetes 社區創建,而是 Docker 公司重構產物
</details>

---

**Q4.** 在 Kubernetes 中,以下哪種調用鏈性能更好？

A) CRI → dockershim → Docker → containerd → 容器
B) CRI → Docker Engine → containerd → 容器
C) CRI → containerd → 容器
D) Docker CLI → Docker Engine → 容器

<details>
<summary>答案</summary>
<b>C) CRI → containerd → 容器</b>

直接使用 CRI 接口調用 containerd 去操作容器,省去了 dockershim 和 Docker Engine 兩個環節,更加簡潔明了,損耗更少,性能也會提升。根據測試數據,containerd 1.1 相比 Docker 18.03,Pod 啟動延遲降低約 20%,CPU 使用率降低 68%,內存使用率降低 12%。

其他選項錯誤原因：
- A: 包含多余的 dockershim 和 Docker Engine 環節
- B: 不符合 CRI 接口調用方式
- D: 不涉及 Kubernetes 的調用鏈
</details>

---

**Q5.** Kubernetes 正式宣布棄用 Docker 支持是在哪個版本？

A) Kubernetes 1.20 (2020年)
B) Kubernetes 1.23 (2021年)
C) Kubernetes 1.24 (2022年)
D) Kubernetes 1.25 (2022年)

<details>
<summary>答案</summary>
<b>A) Kubernetes 1.20 (2020年)</b>

在 2020 年,Kubernetes 1.20 正式向 Docker「宣戰」,聲明 kubelet將棄用 Docker 支持,並會在未來的版本中徹底刪除。但實際完全刪除是在 1.24 版本(2022年5月)。

其他選項錯誤原因：
- B: 1.23 版本仍未能移除 dockershim,推遲了半年
- C: 1.24 版本是完全刪除 dockershim 的版本,不是宣布棄用的版本
- D: 不符合歷史時間線
</details>

---

**Q6.** Kubernetes 1.24 版本完全移除 dockershim 代碼的時間是？

A) 2020年12月
B) 2021年5月
C) 2022年5月
D) 2022年12月

<details>
<summary>答案</summary>
<b>C) 2022年5月</b>

Kubernetes 原本打算用一年時間完成棄用工作,但因低估 Docker 的根基,1.23版本仍未能移除,推遲半年後,終於在 2022年5月發布的 1.24 版本把 dockershim 代碼從 kubelet 里刪掉。

其他選項錯誤原因：
- A: 2020年12月是宣布棄用不久的時間
- B: 2021年5月是原計劃完成時間
- D: 2022年12月不符合實際發布時間
</details>

---

**Q7.** 「棄用 Docker」後,以下哪項表述是正確的？

A) Docker 鏡像將無法在 Kubernetes 中使用
B) 所有現有的容器都會停止運行
C) Kubernetes 直接調用 containerd,繞過了 Docker Engine
D) 必須重新構建所有應用鏡像才能使用

<details>
<summary>答案</summary>
<b>C) Kubernetes 直接調用 containerd,繞過了 Docker Engine</b>

Kubernetes 只是棄用了 dockershim,直接調用 Docker 内部的 containerd。由於容器鏡像格式已被標準化(OCI規範),Docker鏡像仍然可以在 Kubernetes 里正常使用,原有的開發測試、CI/CD 流程都不需要改動。

其他選項錯誤原因：
- A: 鏡像格式已標準化(OCI),仍可正常使用
- B: 容器繼續正常运行,只是管理方式改變
- D: 不需要重新構建鏡像
</details>

---

**Q8.** Docker 公司接管 dockershim 代碼後建立的項目名稱是？

A) docker-runtime
B) cri-dockerd
C) kubernetes-docker
D) docker-cri-adapter

<details>
<summary>答案</summary>
<b>B) cri-dockerd</b>

Docker 公司把 dockershim 的代碼接管過來,另建了一個叫 cri-dockerd 的項目,作用與dockershim一樣,把 Docker Engine 適配成 CRI 接口,使kubelet 可以通過它來操作 Docker。

其他選項錯誤原因：
- A: 不是 Docker 建立的項目名稱
- C: 不是項目名稱
- D: 不是項目名稱
</details>

---

## 第二部分：是非題 (共5題,每題2分)

**Q9.** Kubernetes 棄用 Docker 意味著 Docker 軟體產品將完全無法使用。 _(True/False)_

<details>
<summary>答案</summary>
<b>False</b> - Kubernetes 棄用的只是 dockershim 這個小組件,也就是把 dockershim 移出了 kubelet,並不是「棄用了 Docker」這個軟體產品。Docker 鏡像和容器仍然會正常运行,Kubernetes 只是繞過了 Docker,直接調用 Docker 内部的 containerd。
</details>

---

**Q10.** containerd 1.1 相比 Docker 18.03 在性能上有顯著提升,Pod 啟動延遲降低了約 20%。 _(True/False)_

<details>
<summary>答案</summary>
<b>True</b> - 根據 Kubernetes 在 2018年發布的博客文章和測試數據,containerd 1.1 相比 Docker 18.03,Pod 啟動延遲降低了約 20%,CPU 使用率降低了 68%,內存使用率降低了 12%,這是一個相當大的性能改善。
</details>

---

**Q11.** Kubernetes 棄用 Docker 後,使用 `docker ps` 命令仍然可以看到 Kubernetes 里運行的容器。 _(True/False)_

<details>
<summary>答案</summary>
<b>False</b> - 如果 Kubernetes 直接使用 containerd 来操縱容器,那麼它就是一個與 Docker 独立的工作環境,彼此都不能訪問对方管理的容器和鏡像。使用命令 `docker ps` 就看不到在 Kubernetes 里運行的容器了。需要使用新的工具 crictl 来查看 Kubernetes 管理的容器。
</details>

---

**Q12.** Docker Desktop 内置了 Kubernetes 環境。 _(True/False)_

<details>
<summary>答案</summary>
<b>True</b> - Docker 是一個完整的軟體產品線,不止是 containerd,它還包括了鏡像構建、分发、測試等許多服務,甚至在 Docker Desktop 里還内置了 Kubernetes。
</details>

---

**Q13.** OCI (Open Container Initiative) 是容器鏡像格式的標準化規範。 _(True/False)_

<details>
<summary>答案</summary>
<b>True</b> - OCI (Open Container Initiative) 是容器鏡像格式的標準化規範。因為容器鏡像格式已被標準化,Docker 鏡像仍然可以在 Kubernetes 里正常使用,原來的開發測試、CI/CD 流程都不需要改動。
</details>

---

## 第三部分：填充題 (共4題,每題3分)

**Q14.** Kubernetes 在 2016年底的 1.5 版本引入了新的接口標準 ________,採用 ProtoBuffer 和 gRPC,規定 kubelet 該如何調用容器運行時去管理容器和鏡像。

<details>
<summary>答案</summary>
<b>CRI (Container Runtime Interface)</b>

CRI 是 Container Runtime Interface 的縮寫,在 Kubernetes 1.5版本引入,標誌著 Kubernetes 開始與 Docker 解耦。
</details>

---

**Q15.** Docker 公司采取「________」策略,推動自身重構,把原本單體架構的 Docker Engine 拆分成多個模塊。

<details>
<summary>答案</summary>
<b>斷臂求生</b>

面對 Kubernetes「咄咄逼人」的架勢,Docker 公司采取「斷臂求生」策略,將 Docker Engine 重構並捐獻部分給 CNCF 形成containerd。
</details>

---

**Q16.** Kubernetes 在 kubelet 和 Docker 中間加入的「適配器」被形象地稱為「________」,意思是「墊片」。

<details>
<summary>答案</summary>
<b>shim</b>

這個「適配器」夾在 kubelet 和 Docker 之間,所以被形象地稱為「shim」,也就是「墊片」的意思,正式名稱是 dockershim。
</details>

---

**Q17.** containerd 1.1 相比 Docker 18.03,CPU 使用率降低了 ________,內存使用率降低了 ________。

<details>
<summary>答案</summary>
<b>68%, 12%</b>

根據 Kubernetes 在 2018年5月發布的博客文章,containerd 1.1 的性能測試數據顯示: Pod 啟動延遲降低約 20%,CPU 使用率降低 68%,內存使用率降低 12%。
</details>

---

## 第四部分：簡答題 (共2題,每題5分)

**Q18.** 請說明 Kubernetes 棄用 Docker 的真正含義,以及對開發者和運維人员的實際影響。

<details>
<summary>答案</summary>
<b>模型答案:</b>

Kubernetes 棄用 Docker 的真正含義是「棄用了 dockershim」這個小組件,而不是「棄用了 Docker」這個軟體產品。實際上,Kubernetes 只是把 dockershim 移出了 kubelet,繞過了 Docker Engine,直接調用 Docker 内部的 containerd。

<b>對開發者和運維人员的實際影響:</b>

1. **鏡像方面:** Docker 鏡像仍然可以在 Kubernetes 中正常使用,因為鏡像格式已標準化(OCI規範),不需要重新構建或修改鏡像。

2. **開發流程:** 原有的開發測試、CI/CD 流程都不需要改動,仍然可以使用 Dockerfile 打包應用,從 Docker Hub 拉取鏡像。

3. **容器管理:** 使用 `docker ps` 命令看不到 Kubernetes 管理的容器,需要改用 crictl 工具查看,但命令子命令相似(ps、images 等),適應難度不大。

4. **開發環境:** Docker 仍然是容器開發的便利工具,Docker Desktop 甚至内置了 Kubernetes,開發者可以繼續在熟悉的 Docker 環境中工作。

<b>關鍵點:</b> 對於一直使用 kubectl 管理 Kubernetes 的人来说,基本沒有任何影響。
</details>

---

**Q19.** Docker 重構自身並分离出 containerd,這是否算是一種「自掘坟墓」的行为?請闡述你的觀點。

<details>
<summary>答案</summary>
<b>模型答案 (需包含以下關鍵點):</b>

<b>不是「自掘坟墓」,而是「斷臂求生」的明智策略:</b>

1. **被迫的選擇:** Docker 公司體量太小,無法與大公司抗衡,Kubernetes 加入 CNCF 并制定 CRI 標準,明顯要與 Docker 解耦。如果 Docker 不主動適應,可能會被完全拋棄。

2. **保持影響力:** 通過捐獻 containerd 給 CNCF,Docker 讓自己的技術成為業界標準的一部分。containerd 符合 CRI 標準,仍然與 Docker 有關聯,保住了 Docker 在容器技術生態中的地位。

3. **延續生命力:** Docker 鏡像格式標準化(OCI),大量鏡像和忠实用戶是 Docker 的最大資本。即使 Kubernetes 不直接使用 Docker,Docker 仍然是開發者的首選工具。

4. **多元化發展:** Docker 不止是 containerd,還包括鏡像構建、分发、測試等服務,Docker Desktop 内置 Kubernetes,展现了 Docker 的產品完整性和生態適應能力。

<b>如果不分离 containerd 的后果:</b>

- Kubernetes 可能完全拋棄 Docker,開發其他運行時(如 CRI-O)
- Docker 在雲原生生態中會被邊缘化甚至淘汰
- 失去與 Kubernetes 的任何關聯,市場份額急劇下降

<b>結論:</b> Docker 的策略是「一时的退让是为了更好的将来」,將 containerd 分離並標準化,反而讓 Docker 在新格局中找到了生存之道。
</details>

---

## 第五部分：配對題 (共1題,5分)

**Q20.** 將以下術語與其描述進行配對:

| 術語 | 描述 |
| --- | --- |
| 1. CRI | A. Docker 公司接管 dockershim 建立的項目,將 Docker Engine 適配成 CRI 接口 |
| 2. containerd | B. Kubernetes 與 Docker 之間的適配器,將 Docker 接口轉換成 CRI標準 |
| 3. dockershim | C. Container Runtime Interface,Kubernetes 定義的容器運行時接口標準 |
| 4. OCI | D. Docker Engine 重構後捐獻給 CNCF 的部分,符合 CRI 標準的容器運行時 |
| 5. cri-dockerd | E. Open Container Initiative,容器鏡像格式的標準化規範 |

<details>
<summary>答案</summary>
<b>配對關係:</b>

<b>1-C</b> (CRI 是 Container Runtime Interface,Kubernetes 定義的接口標準)
<b>2-D</b> (containerd 是 Docker捐獻給 CNCF 的容器運行時)
<b>3-B</b> (dockershim 是 kubelet 和 Docker之間的適配器)
<b>4-E</b> (OCI 是容器鏡像格式標準化規範)
<b>5-A</b> (cri-dockerd 是 Docker 公司建立的項目,將 Docker Engine 適配成 CRI 接口)

<b>完整配對:</b> 1-C, 2-D, 3-B, 4-E, 5-A
</details>

---

## 答案總覽

### 第一部分:選擇題
1. B - Kubernetes 1.5 (2016年)
2. B - 作為適配器轉換接口
3. B - Docker 重構並捐獻給 CNCF
4. C - CRI → containerd → 容器
5. A - Kubernetes 1.20 (2020年)
6. C - 2022年5月
7. C - 直接調用 containerd
8. B - cri-dockerd

### 第二部分:是非題
9. False - 只是棄用 dockershim,不是 Docker 軟體
10. True - 性能確有顯著提升
11. False - docker ps 看不到 K8s容器
12. True - Docker Desktop 内置 Kubernetes
13. True - OCI 是鏡像格式標準

### 第三部分:填充題
14. CRI (Container Runtime Interface)
15. 斷臂求生
16. shim
17. 68%, 12%

### 第四部分:簡答題
18. 請參考詳細答案
19. 請參考詳細答案

### 第五部分:配對題
20. 1-C, 2-D, 3-B, 4-E, 5-A

---

**測驗生成時間:** 2026-05-11
**來源資料:** 加餐｜Kubernetes「棄用 Docker」是怎麼回事？.pdf
**課程:** Kubernetes 入門課程

_此測驗涵蓋了 Kubernetes 棄用 Docker 的歷史背景、技術原理、實際影響和未來展望,幫助學員深入理解容器技術生態的演變過程。_