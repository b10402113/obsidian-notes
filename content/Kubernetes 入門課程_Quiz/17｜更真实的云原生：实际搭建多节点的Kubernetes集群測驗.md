# 多節點 Kubernetes 集群搭建測驗

**科目:** Kubernetes 入門課程
**主題:** 17｜更真實的雲原生：實際搭建多節點的 Kubernetes 集群
**難度:** 中等
**題目數:** 15
**建議時間:** 30-40 分鐘

---

## 第一部分：選擇題 (40分)

**Q1.** kubeadm 的主要用途是什麼？

A) 在單機環境中創建 Kubernetes 集群，類似 minikube
B) 在集群環境中部署接近生產級質量的 Kubernetes
C) 監控 Kubernetes 集群的運行狀態
D) 編寫 Kubernetes 的配置文件

<details>
<summary>答案</summary>
<b>B)</b> kubeadm 的目標是在集群環境中輕鬆部署 Kubernetes，並使其接近甚至達到生產級質量。雖然它和 minikube 都使用容器和鏡像來封裝 Kubernetes 的各種組件，但 kubeadm 針對的是多節點集群部署，而 minikube 主要用於單機環境的學習和測試。

<b>為何其他選項錯誤：</b>
- A: minikube 才是專注於單機環境的工具
- C: kubeadm 是部署工具，不是監控工具
- D: kubeadm 是部署工具，不是配置文件編寫工具
</details>

---

**Q2.** 在實驗環境架構中，Master 節點與 Worker 節點的主要區別是什麼？

A) Master 節點運行業務應用，Worker 節點運行管理組件
B) Master 節點運行 apiserver、etcd、scheduler 等管理組件，Worker 節點運行業務應用
C) Master 節點配置要求較低，Worker 節點配置要求較高
D) Master 節點不需要網絡配置，Worker 節點需要網絡配置

<details>
<summary>答案</summary>
<b>B)</b> Master 節點需要運行 apiserver、etcd、scheduler、controller-manager 等組件來管理整個集群，而 Worker 節點沒有管理工作，只運行業務應用。

<b>為何其他選項錯誤：</b>
- A: 職責分配正好相反
- C: Master 節點因需要運行管理組件，配置要求至少是 2核CPU、4GB記憶體，反而比 Worker 節點要求更高
- D: 所有節點都需要網絡配置
</details>

---

**Q3.** Kubernetes 安裝前需要做的四項準備工作中，為什麼要關閉 Linux 的 swap 分區？

A) 為了節省磁盤空間
B) 為了提升 Kubernetes 的性能
C) 為了避免與 Docker 衝突
D) 為了簡化網絡配置

<details>
<summary>答案</summary>
<b>B)</b> 關閉 Linux 的 swap 分區是為了提升 Kubernetes 的性能。如果不關閉 swap，kubelet 甚至無法正常啟動。

<b>為何其他選項錯誤：</b>
- A: 雖然關閉 swap 確實會釋放一些磁盤空間，但這不是 Kubernetes 要求關閉的主要原因
- C: 關閉 swap 不是為了避免與 Docker 衝突
- D: swap 分區與網絡配置無關
</details>

---

**Q4.** Kubernetes 的組件鏡像存放在哪個鏡像倉庫？

A) Docker Hub
B) Alibaba Cloud Container Registry
C) gcr.io (Google Container Registry)
D) quay.io

<details>
<summary>答案</summary>
<b>C)</b> Kubernetes 的組件鏡像存放在 Google 自己的鏡像倉庫網站 gcr.io。這在中國的訪問很困難，需要採取變通措施提前把鏡像下載到本地。

<b>為何其他選項錯誤：</b>
- A: Docker Hub 不存放 Kubernetes 官方組件鏡像
- B: Alibaba Cloud 是國內的替代方案，不是官方存放位置
- D: quay.io 不是 Kubernetes 官方組件鏡像倉庫
</details>

---

**Q5.** 使用 kubeadm init 安裝 Master 節點時，參數 `--pod-network-cidr=10.10.0.0/16` 的作用是什麼？

A) 設定 apiserver 的 IP 地址
B) 設定集群裡 Pod 的 IP 地址段
C) 設定 Master 節點的 IP 地址
D) 設定 Worker 節點的 IP 地址段

<details>
<summary>答案</summary>
<b>B)</b> `--pod-network-cidr` 參數用來設置集群裡 Pod 的 IP 地址段。這個地址段需要與 Flannel 網絡插件的配置一致。

<b>為何其他選項錯誤：</b>
- A: apiserver 的 IP 地址由 `--apiserver-advertise-address` 參數設定
- C: Master 節點的 IP 地址是主機本身的 IP，由網絡配置決定
- D: Worker 節點沒有獨立的 IP 地址段設置
</details>

---

**Q6.** Master 節點安裝完成後，為什麼節點狀態會顯示為 "NotReady"？

A) apiserver 未啟動
B) kubelet 配置錯誤
C) 缺少網絡插件，集群內部網絡尚未正常運作
D) Docker 未正常運行

<details>
<summary>答案</summary>
<b>C)</b> Master 節點安裝完成後狀態顯示為 "NotReady"，是因為還缺少網絡插件，集群的內部網絡還沒有正常運作。需要部署 Flannel 等網絡插件後才能正常工作。

<b>為何其他選項錯誤：</b>
- A: 如果 apiserver 未啟動，kubeadm init 會報錯，不會成功完成
- B: kubelet 配置由 kubeadm 管理，通常不會出錯
- D: Docker 是 Kubernetes 的基礎，如果未運行，安裝過程中就會報錯
</details>

---

**Q7.** 安裝 Flannel 網絡插件時，需要修改 `kube-flannel.yml` 文件中的哪個字段？

A) apiVersion
B) kind
C) net-conf.json 中的 Network 字段
D) metadata.name

<details>
<summary>答案</summary>
<b>C)</b> 需要修改 `net-conf.json` 字段中的 Network，將其改成 kubeadm init 時 `--pod-network-cidr` 參數設置的地址段，例如 "10.10.0.0/16"。

<b>為何其他選項錯誤：</b>
- A, B, D: 這些字段不需要修改，只需修改網絡配置相關的 Network 字段
</details>

---

**Q8.** Worker 節點加入 Kubernetes 集群需要使用哪個命令？

A) kubeadm init
B) kubeadm join
C) kubeadm upgrade
D) kubeadm reset

<details>
<summary>答案</summary>
<b>B)</b> Worker 節點需要使用 `kubeadm join` 命令來加入集群。這個命令在 Master 節點安裝完成後會顯示出來，包含 token 和 ca 證書信息。

<b>為何其他選項錯誤：</b>
- A: kubeadm init 用於安裝 Master 節點
- C: kubeadm upgrade 用於升級 Kubernetes 版本
- D: kubeadm reset 用於重置節點狀態
</details>

---

## 第二部分：是非題 (20分)

**Q9.** minikube 和 kubeadm 都使用容器和鏡像來封裝 Kubernetes 的各種組件。 _(對/錯)_

<details>
<summary>答案</summary>
<b>對</b> - kubeadm 的原理和 minikube 類似，都是用容器和鏡像來封裝 Kubernetes 的各種組件，如 apiserver、etcd、scheduler 等。但 kubeadm 的目標是集群部署，而 minikube 是單機部署。
</details>

---

**Q10.** Console 節點必須是一台獨立的物理服務器或虛擬機。 _(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - Console 只是一個邏輯概念，不一定要是獨立的服務器。在實際部署時完全可以復用之前 minikube 的虛擬機，或者直接使用 Master/Worker 節點作為控制台。
</details>

---

**Q11.** 在 Kubernetes 集群中，每個節點的主機名 (hostname) 可以相同。 _(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - 由於 Kubernetes 使用主機名來區分集群裡的節點，所以每個節點的 hostname 必須不能重名。需要修改 `/etc/hostname` 文件，為每個節點設置容易辨識的獨特名字。
</details>

---

**Q12.** Kubernetes 目前支持多種容器運行時，但 Docker 是最方便最易用的一種。 _(對/錯)_

<details>
<summary>答案</summary>
<b>對</b> - Kubernetes 確實支持多種容器運行時（如 containerd、CRI-O 等），但課程中仍然使用 Docker 作為 Kubernetes 的底層支持，因為它是最方便最易用的一種。
</details>

---

**Q13.** 安裝完 Flannel 網絡插件後，Master 節點狀態應該變為 "Ready"。 _(對/錯)_

<details>
<summary>答案</summary>
<b>對</b> - 安裝 Flannel 網絡插件並等鏡像拉取下來運行之後，Master 節點狀態會從 "NotReady" 變為 "Ready"，表明節點網絡工作正常了。
</details>

---

## 第三部分：填空題 (20分)

**Q14.** kubeadm 是一個專門用來在集群中安裝 Kubernetes 的工具，它的名字含義是 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>Kubernetes 管理員</b>
</details>

---

**Q15.** 安裝 Kubernetes 前需要修改 Docker 配置，在 `/etc/docker/daemon.json` 中要把 cgroup 的驅動程序改成 **\_\_\_\_**。

<details>
<summary>答案</summary>
<b>systemd</b>
</details>

---

**Q16.** Kubernetes 集群的架構中，Master 節點的配置要求至少是 **\_\_\_\_** CPU 和 **\_\_\_\_** 記憶體。

<details>
<summary>答案</summary>
<b>2核, 4GB</b>
</details>

---

**Q17.** 使用命令 **\_\_\_\_** 可以查看安裝 Kubernetes 所需的鏡像列表。

<details>
<summary>答案</summary>
<b>kubeadm config images list</b>
</details>

---

**Q18.** Worker 節點加入集群時，kubeadm join 命令需要包含 **\_\_\_\_** 和 **\_\_\_\_** 這兩個重要信息。

<details>
<summary>答案</summary>
<b>token, ca 證書 (discovery-token-ca-cert-hash)</b>
</details>

---

## 第四部分：簡答題 (20分)

**Q19.** 說明在安裝 Kubernetes 前需要做的四項準備工作及其作用。

<details>
<summary>答案</summary>
安裝 Kubernetes 前需要在 Master 和 Worker 節點上做四項準備工作：

<b>1. 修改主機名</b>
修改 `/etc/hostname` 文件，為每個節點設置獨特的名字（如 master、worker），因為 Kubernetes 使用主機名來區分集群裡的節點。

<b>2. 修改 Docker 配置</b>
在 `/etc/docker/daemon.json` 中將 cgroup 驅動程序改成 systemd，確保 Docker 和 Kubernetes 的 cgroup 管理方式一致。

<b>3. 修改網絡設置</b>
啟用 br_netfilter 模組，修改 iptables 配置，讓 Kubernetes 能夠檢查和转发網絡流量。

<b>4. 關閉交換分區</b>
修改 `/etc/fstab` 關閉 Linux 的 swap 分區，提升 Kubernetes 性能。如果不關閉 swap，kubelet 甚至無法啟動。
</details>

---

**Q20.** 解釋為什麼在中國需要採取變通措施來獲取 Kubernetes 組件鏡像，並說明兩種獲取方法。

<details>
<summary>答案</summary>
<b>原因：</b>
Kubernetes 的組件鏡像（如 kube-apiserver、etcd、scheduler 等）存放在 Google 自己的鏡像倉庫 gcr.io，在中國訪問很困難，直接拉取鏡像幾乎不可能。

<b>兩種獲取方法：</b>

<b>方法一：利用 minikube</b>
從 minikube 節點中導出鏡像。具體做法：
1. 啟動 minikube
2. 使用 minikube ssh 登錄進虛擬節點
3. 用 docker save -o 命令保存相應版本的鏡像
4. 用 minikube cp 拷貝到本地
5. 在目標節點上用 docker load 加載

這種方法安全可靠，但操作較麻煩。

<b>方法二：從國內鏡像網站下載</b>
使用 Shell 腳本從國內鏡像倉庫（如 registry.aliyuncs.com/google_containers）下載，再用 docker tag 改名。

這種方法速度快，但有隱患（網站可能停止服務或改動鏡像）。建議結合兩種方法，先用腳本下載，再用 minikube 鏡像做對比驗證。
</details>

---

**Q21.** 說明 Console 節點的作用以及它與 Kubernetes 群的關係。

<details>
<summary>答案</summary>
<b>Console 節點的作用：</b>
Console 節點是一台起輔助作用的服務器，主要功能是在上面安裝命令行工具 kubectl。所有對 Kubernetes 集群的管理命令都從這台主機發出。

<b>與集群的關係：</b>
1. Console 節點不在 Kubernetes 集群內部，它是一台外部的管理主機
2. 它只需要安裝 kubectl，然後複製 kubeconfig 文件（`~/.kube/config`）
3. 基於安全考慮，集群裡的主機部署好後應盡量少直接登錄操作，所以需要從外部進行管理
4. Console 只是邏輯概念，可以是獨立的服務器，也可以復用 minikube 虛擬機或直接使用 Master/Worker 節點作為控制台

<b>設置方式：</b>
可以在 Master 節點上使用 scp 命令將 kubectl 和 config 文件拷貝到 Console 節點，例如：
```bash
scp `which kubectl` chrono@192.168.10.208:~/
scp ~/.kube/config chrono@192.168.10.208:~/.kube
```
</details>

---

## 第五部分：配對題 (20分)

**Q22.** 將下列 kubeadm 命令與其功能進行配對：

| 命令             | 功能                         |
| ---------------- | ---------------------------- |
| 1. kubeadm init  | A. 升級 Kubernetes 版本      |
| 2. kubeadm join  | B. 重置節點狀態              |
| 3. kubeadm upgrade | C. 安裝 Master 節點        |
| 4. kubeadm reset | D. Worker 節點加入集群       |

<details>
<summary>答案</summary>
1-C, 2-D, 3-A, 4-B

<b>解釋：</b>
- kubeadm init: 在 Master 節點上初始化 Kubernetes 集群
- kubeadm join: Worker 節點使用此命令加入已存在的集群
- kubeadm upgrade: 用於升級 Kubernetes 集群版本
- kubeadm reset: 重置節點，清除 Kubernetes 安裝狀態
</details>

---

**Q23.** 將下列 Kubernetes 組件與其功能進行配對：

| 組件              | 功能                           |
| ----------------- | ------------------------------ |
| 1. apiserver      | A. 存儲集群配置數據            |
| 2. etcd           | B. 調度 Pod 到節點             |
| 3. scheduler      | C. 提供 API 接口               |
| 4. controller-manager | D. 管理控制器運行邏輯      |

<details>
<summary>答案</summary>
1-C, 2-A, 3-B, 4-D

<b>解釋：</b>
- apiserver: Kubernetes 的 API 服務器，提供 REST API 接口
- etcd: 高可用鍵值存儲，保存集群的所有配置和狀態數據
- scheduler: 諸調度器，負責將 Pod 諸調度到合適的節點上運行
- controller-manager: 運行各種控制器，管理集群的運行邏輯
</details>

---

**Q24.** 將下列節點類型與其特點進行配對：

| 節點類型 | 特點                               |
| -------- | ---------------------------------- |
| 1. Master | A. 配置要求較低，可為 1核1GB     |
| 2. Worker | B. 需運行 apiserver 等管理組件    |
| 3. Console | C. 不在集群內，作為管理控制台    |

<details>
<summary>答案</summary>
1-B, 2-A, 3-C

<b>解釋：</b>
- Master 節點: 需運行 apiserver、etcd、scheduler、controller-manager 等管理組件，配置要求至少 2核CPU、4GB記憶體
- Worker 節點: 只運行業務應用，沒有管理工作，配置要求較低，最低可為 1核CPU、1GB記憶體
- Console 節點: 不是集群節點，只是外部的管理控制台，用於運行 kubectl 命令管理集群
</details>

---

## 答案總覽

### 第一部分：選擇題
1. B
2. B
3. B
4. C
5. B
6. C
7. C
8. B

### 第二部分：是非題
9. 
10. 錯
11. 錯
12. 對
13. 對

### 第三部分：填空題
14. Kubernetes 管理員
15. systemd
16. 2核, 4GB
17. kubeadm config images list
18. token, ca 證書 (discovery-token-ca-cert-hash)

### 第四部分：簡答題
19-21. 見各題詳細答案

### 第五部分：配對題
22. 1-C, 2-D, 3-A, 4-B
23. 1-C, 2-A, 3-B, 4-D
24. 1-B, 2-A, 3-C

---

**生成來源:** 17｜更真實的雲原生：實際搭建多節點的 Kubernetes 集群 - Kubernetes 入門實戰課
**生成時間:** 2026-05-11