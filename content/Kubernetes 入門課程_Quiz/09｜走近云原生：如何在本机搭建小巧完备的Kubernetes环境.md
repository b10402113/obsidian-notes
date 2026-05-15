# 走近云原生：如何在本机搭建小巧完备的Kubernetes环境 - 测验

**科目:** Kubernetes 入門課程
**難度:** 中等
**題數:** 15
**建議時間:** 30 分鐘

---

## 第一部分：選擇題 (每題 5 分，共 25 分)

**Q1.** Kubernetes 的前身是 Google 內部開發的哪個系統？

A) Omega
B) Borg
C) MapReduce
D) BigTable

<details>
<summary>答案</summary>
<b>B) Borg</b>

Google 在公司內部使用了名為 Borg 的集群應用管理系統，後來將其重寫並開源成為 Kubernetes。Omega 是 Borg 的升級版本，MapReduce 和 BigTable 是其他 Google 技術項目。
</details>

---

**Q2.** CNCF（雲原生計算基金會）是在哪一年成立的？

A) 2013 年
B) 2014 年
C) 2015 年
D) 2016 年

<details>
<summary>答案</summary>
<b>C) 2015 年</b>

Google 在 2014 年開源了 Kubernetes，並於 2015 年聯合 Linux 基金會成立了 CNCF，將 Kubernetes 捐獻出來作為種子項目。
</details>

---

**Q3.** 以下哪項不是 Kubernetes 能夠解決的容器管理問題？

A) 服務發現
B) 負載均衡
C) 應用打包
D) 擴容縮容

<details>
<summary>答案</summary>
<b>C) 應用打包</b>

應用打包是 Docker 等容器技術解決的核心問題。Kubernetes 解決的是容器之上的管理、調度工作，包括服務發現、負載均衡、狀態監控、健康檢查、擴容縮容等更高層次的問題。
</details>

---

**Q4.** 在 Kubernetes 中，Pod 可以被理解為：

A) 一個普通的容器
B) 一個虛擬機
C) "穿了馬甲"的容器
D) 一個鏡像文件

<details>
<summary>答案</summary>
<b>C) "穿了馬甲"的容器</b>

Pod 是 Kubernetes 中的重要概念，文中將其形象地比喻為"穿了馬甲"的容器。Pod 不是簡單的容器，而是 Kubernetes 管理的基本單元。
</details>

---

**Q5.** minikube 的最大特點是什麼？

A) 運行速度快，功能少
B) 小而美，集成了 Kubernetes 大多數功能
C) 只支持 Linux 系統
D) 需要大量系統資源

<details>
<summary>答案</summary>
<b>B) 小而美，集成了 Kubernetes 大多數功能</b>

minikube 可執行文件不到 100MB，運行鏡像約 1GB，但卻集成了 Kubernetes 的絕大多數功能特性，包括 Dashboard、GPU、Ingress 等豐富插件，非常完善。
</details>

---

## 第二部分：是非題 (每題 4 分，共 20 分)

**Q6.** Kubernetes 用於解決容器的打包和分發問題。_(對/錯)_


<details>
<summary>答案</summary>
<b>錯</b>

容器技術解決的是應用的打包、分發問題，實現"一次開發，到處運行"。Kubernetes 解決的是容器之上的管理、調度工作，即容器編排問題，面對的是複雜生產環境中的服務發現、負載均衡、擴容縮容等需求。
</details>

---

**Q7.** minikube 和 kind 都是官網推薦的在本機運行 Kubernetes 環境的工具。_(對/錯)_


<details>
<summary>答案</summary>
<b>對</b>

Kubernetes 官網推薦的兩個在本機運行完整 Kubernetes 環境的工具就是 kind 和 minikube。kind 意思是"Kubernetes in Docker"，功能少但速度快；minikube 是迷你版本的 Kubernetes，功能更完善。
</details>

---

**Q8.** kubectl 包含在 minikube 安裝包中，不需要額外安裝。_(對/錯)_


<details>
<summary>答案</summary>
<b>錯</b>

kubectl 是一個與 Kubernetes、minikube彼此獨立的項目，不包含在 minikube裡。但 minikube 提供了安裝它的簡化方式，只需執行 `minikube kubectl` 命令即可下載與當前 Kubernetes 版本匹配的 kubectl。
</details>

---

**Q9.** Kubernetes 在兩年內就成為容器編排領域的唯一霸主。_(對/錯)_


<details>
<summary>答案</summary>
<b>對</b>

由於 Google 和 Linux 基金會的保驾护航，加上寬容開放的社區，Kubernetes 作為 CNCF 的核心項目，仅用了兩年時間就打败了 Apache Mesos 和 Docker Swarm，成為容器編排領域的唯一霸主。
</details>

---

**Q10.** 使用 minikube 啟動 Kubernetes 集群時，必須指定 Kubernetes 版本。_(對/錯)_


<details>
<summary>答案</summary>
<b>錯</b>

使用 `minikube start` 命令會從 Docker Hub 拉取鏡像，以當前最新版本的 Kubernetes 啟動集群。指定版本（使用 `--kubernetes-version`參數）是可选的，目的是保證實驗環境的一致性。
</details>

---

## 第三部分：填空題 (每題 6 分，共 24 分)

**Q11.** 容器技術解決了應用的____和____問題，而 Kubernetes 解決的是容器之上的____工作。

<details>
<summary>答案</summary>
<b>打包、分發、管理調度（或容器編排）</b>

容器技術解決了應用的打包、分發問題，實現"一次開發，到處運行"。Kubernetes 解決的是容器之上的管理、調度工作，即容器編排（Container Orchestration）問題。
</details>

---

**Q12.** minikube 支持的三大主流平台是____、____和____。

<details>
<summary>答案</summary>
<b>Mac、Windows、Linux</b>

minikube 支持 Mac、Windows、Linux這三種主流平台，可以在它的官網找到詳細的安裝說明。
</details>

---

**Q13.** 在 minikube 環境中，會用到兩個客戶端工具：____用於管理 Kubernetes 集群環境，____用於操作實際的 Kubernetes 功能。

<details>
<summary>答案</summary>
<b>minikube、kubectl</b>

minikube 管理 Kubernetes 集群環境，kubectl 操作實際的 Kubernetes 功能。kubectl 的作用類似 docker 命令行工具，是與 Kubernetes後台服務通信的客戶端工具。
</details>

---

**Q14.** Kubernetes 源自 Google 內部的____系統，該系統用____語言開發，後被重寫為____並開源。

<details>
<summary>答案</summary>
<b>Borg、C++、Go語言</b>

Kubernetes 源自 Google 內部的 Borg 系統，該系統用 C++開發。2014 年，Google 用 Go語言重寫並開源成為 Kubernetes。
</details>

---

## 第四部分：簡答題 (每題 10 分，共 20 分)

**Q15.** 請簡述"雲原生"的概念，以及它與 Kubernetes 的關係。

<details>
<summary>答案</summary>
"雲原生"指的是應用的開發、部署、运维等一系列工作都要向 Kubernetes 齊，使用容器、微服務、聲明式 API 等技術，保證應用的整個生命週期都能夠在 Kubernetes 環境裡順利實施，不需要附加額外的條件。

"雲"現在指的是 Kubernetes，"雲原生"的意思就是 Kubernetes 裡的"原住民"，而不是從其他環境迁過來的"移民"。應用需要從設計之初就考慮到在 Kubernetes 上運行的特性，充分利用其提供的功能。

**關鍵點包括：**
- 使用容器技術
- 采用微服務架构
- 使用声明式 API
- 应用整个生命周期都在 Kubernetes 中
</details>

---

**Q16.** 說明 minikube 與 kind 的主要區別，以及為什麼作者建議學習時選擇 minikube。

<details>
<summary>答案</summary>
**主要區別：**

**kind (Kubernetes in Docker):**
- 基於 Docker
- 功能少，用法簡單
- 運行速度快，容易上手
- 缺少很多 Kubernetes 的標準功能（如仪表盘、网络插件）
- 很難定制化
- 名稱與 Kubernetes YAML 配置的字段 kind 重名，容易混淆

**minikube:**
- 迷你版本的 Kubernetes
- 自2016年發布以来一直積極開發維護
- 緊跟 Kubernetes 版本更新
- 小而美：可執行文件不到 100MB，運行鏡像約 1GB
- 集成了 Kubernetes 大多數功能特性
- 有豐富插件（Dashboard、GPU、Ingress、Istio、Kong、Registry等）

**建議選擇 minikube 的原因：**
1. 功能更完善，适合學習研究
2. 避免名字混淆，不干擾學習
3. 雖小但功能齊全，能全面了解 Kubernetes
</details>

---

## 第五部分：配對題 (11 分)

**Q17.** 将以下術語與其定義進行配對：

| 術語 | 定義 |
|------|------|
| 1. 容器編排 | A. Kubernetes 管理的基本單元，可以理解為"穿了馬甲"的容器 |
| 2. Kubernetes | B. 容器之上的管理、調度工作，組織管理各個應用容器之間的關係 |
| 3. kubectl | C. 生產級別的容器编排平台和集群管理系统 |
| 4. Pod | D. 在本機搭建 Kubernetes 環境的工具，小而美 |
| 5. minikube | E. 操作 Kubernetes 的命令行客戶端工具 |

<details>
<summary>答案</summary>
1-B, 2-C, 3-E, 4-A, 5-D

<b>配對說明：</b>
- **容器編排 (B)**: 管理調度容器之間關係的工作
- **Kubernetes (C)**: 容器编排平台和集群管理系统
- **kubectl (E)**: 命令行工具，與 Kubernetes 后台服務通信
- **Pod (A)**: Kubernetes 基本管理單元
- **minikube (D)**: 本機 Kubernetes 環境工具
</details>

---

## 答案總結

### 選擇題
1. B) Borg
2. C) 2015 年
3. C) 應用打包
4. C) "穿了馬甲"的容器
5. B) 小而美，集成了 Kubernetes 大多數功能

### 是非題
6. 錯 - 容器技術解決打包分發，Kubernetes 解決容器編排
7. 對 - kind 和 minikube 都是官網推薦工具
8. 錯 - kubectl 是獨立項目，需額外安裝
9. 對 - 用了兩年成為唯一霸主
10. 錯 - 指定版本是可選的

### 填空題
11. 打包、分發、管理調度
12. Mac、Windows、Linux
13. minikube、kubectl
14. Borg、C++、Go語言

### 簡答題
15. 雲原生指應用全生命週期在 Kubernetes 中實施，使用容器、微服務、声明式 API 等技術
16. minikube 功能完善適合學習，kind 功能少速度快適合測試；建議 minikube 因功能齐全且不混淆

### 配對題
17. 1-B, 2-C, 3-E, 4-A, 5-D

---

_生成來源: Kubernetes 入門課程第 09 篇_