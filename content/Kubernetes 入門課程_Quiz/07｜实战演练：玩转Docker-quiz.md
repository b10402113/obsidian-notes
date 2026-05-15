# Docker實戰演練測驗

**科目:** Kubernetes入門課程 - 第07課
**難度:** 混合（基礎、中等、進階）
**題數:** 25題
**建議時間:** 30-45分鐘

---

## 第一部分：選擇題（15題，每題4分）

**Q1.** 容器技術是基於Linux底層的哪些功能實現的？

A) namespace、cgroup、chroot
B) hypervisor、VM、kernel
C) systemd、init、daemon
D) socket、pipe、signal

<details>
<summary>答案</summary>
<b>A)</b> 容器技術基於Linux底層的namespace（命名空間）、cgroup（控制組）和chroot等功能實現資源隔離。Docker將這些技術整合在一起，使容器真正走近大眾視野。
</details>

---

**Q2.** 容器技術的三個核心概念是什麼？

A) Container、Process、Thread
B) Container、Image、Registry
C) Container、Volume、Network
D) Container、Pod、Service

<details>
<summary>答案</summary>
<b>B)</b> 容器技術的三個核心概念是：容器（Container）、鏡像（Image）和鏡像倉庫（Registry）。這三者構成了容器技術的基礎架構。
</details>

---

**Q3.** 關於容器與虛擬機的比較，下列哪項描述正確？

A) 虛擬機比容器更輕量級，運行效率更高
B) 容器需要完整的操作系統，虛擬機不需要
C) 容器更加輕量級，運行效率更高，比虛擬機更適合雲計算需求
D) 容器和虛擬機都需要硬件層級的虛擬化

<details>
<summary>答案</summary>
<b>C)</b> 容器屬於虛擬化技術的一種，和虛擬機都能分拆系統資源、隔離應用進程，但容器更加輕量級，運行效率更高，比虛擬機更適合雲計算的需求。
</details>

---

**Q4.** 鏡像內部由多個層（Layer）組成，這些層使用什麼技術合併成一個文件系統？

A) RAID技術
B) Union FS技術
C) NFS技術
D) EXT4技術

<details>
<summary>答案</summary>
<b>B)</b> 鏡像內部由多個層組成，每一層都是一組文件，多個層會使用Union FS（Union File System）技術合併成一個文件系統供容器使用。這種結構允許相同的層共享、復用，節約存儲和傳輸成本。
</details>

---

**Q5.** Docker Registry容器的內部端口號是多少？

A) 80端口
B) 5000端口
C) 3306端口
D) 443端口

<details>
<summary>答案</summary>
<b>B)</b> Docker Registry容器內部使用5000端口提供服務，通常會將宿主機的5000端口映射到容器的5000端口。
</details>

---

**Q6.** 在搭建WordPress網站時，MariaDB容器的端口號是多少？

A) 80端口
B) 5000端口
C) 3306端口
D) 9000端口

<details>
<summary>答案</summary>
<b>C)</b> MariaDB作為關系型數據庫，其標準端口號是3306。在容器環境中，這個端口被容器隔離，外界不可見。
</details>

---

**Q7. Docker的bridge網絡模式的默認網段是什麼？

A) 192.168.0.0/16
B) 10.0.0.0/8
C) 172.17.0.0/16
D) 172.16.0.0/12

<details>
<summary>答案</summary>
<b>C)</b> Docker的bridge網絡模式的默認網段是172.17.0.0/16，宿主機固定是172.17.0.1，IP地址是順序分配的。
</details>

---

**Q8.** 要查看容器IP地址，應該使用哪個命令？

A) docker ps
B) docker inspect
C) docker exec
D) docker logs

<details>
<summary>答案</summary>
<b>B)</b> 使用docker inspect命令可以查看容器的詳細信息，包括IP地址。例如：docker inspect [container_id] | grep IPAddress。
</details>

---

**Q9.** 將鏡像推送到私有倉庫時，鏡像名稱前面必須加上什麼？

A) 鏡像的SHA值
B) 鏡像的大小信息
C) 倉庫的地址（域名或IP地址）
D) 鏡像的創建時間

<details>
<summary>答案</summary>
<b>C)</b> 因為推送的目標不是默認的Docker Hub，而是本地的私有倉庫，所以鏡像的名字前面必須再加上倉庫的地址（域名或者IP地址）。例如：127.0.0.1:5000/nginx:alpine。
</details>

---

**Q10.** 在WordPress容器中，WORDPRESS_DB_HOST環境變量應該設置為什麼？

A) localhost
B) 127.0.0.1
C) MariaDB容器的IP地址
D) Docker Hub的地址

<details>
<summary>答案</summary>
<b>C)</b> WORDPRESS_DB_HOST必須是MariaDB的IP地址（例如172.17.0.2），否則WordPress無法連接到數據庫。這是因為容器之間通過bridge網絡通信。
</details>

---

**Q11.** 在WordPress架構中，Nginx的角色是什麼？

A) 數據庫服務器
B) 應用服務器
C) 反向代理，將請求转发給WordPress
D) 鏡像倉庫

<details>
<summary>答案</summary>
<b>C)</b> Nginx是前面的反向代理，它對外暴露80端口，然後把請求转发給WordPress應用服務器。WordPress本身的80端口沒有映射到外部。
</details>

---

**Q12.** 使用docker exec連接到MariaDB容器時，使用了哪個客戶端工具？

A) postgres
B) sqlite
C) mysql
D) mongo

<details>
<summary>答案</summary>
<b>C)</b> MariaDB是MySQL的分支，使用mysql客戶端工具連接。命令示例：docker exec -it [container_id] mysql -u wp -p。
</details>

---

**Q13.** Docker Registry提供了什麼方式查看倉庫里的鏡像？

A) GUI圖形界面
B) RESTful API
C) SSH連接
D) FTP服務

<details>
<summary>答案</summary>
<b>B)</b> Docker Registry雖然沒有圖形界面，但提供了RESTful API，可以發送HTTP請求來查看倉庫里的鏡像。例如：curl 127.1:5000/v2/_catalog。
</details>

---

**Q14.** 127.0.0.1可以簡寫成什麼形式？

A) 127.0.0
B) 127.1
C) 127
D) localhost

<details>
<summary>答案</summary>
<b>B)</b> 127.0.0.1可以簡寫成127.1，因為中間的0可以壓縮。這是一種通用的簡寫方式，在curl命令中可以使用。
</details>

---

**Q15.** 文中提到的容器編排技術主要用於解決什麼問題？

A) 增加鏡像大小
B) 管理容器的運行次序、網絡連接、數據持久化等應用要素
C) 提高單個容器的性能
D) 增加容器的數量上限

<details>
<summary>答案</summary>
<b>B)</b> 容器編排是在更高的层次上規劃容器的運行次序、網絡連接、數據持久化等應用要素，解決多容器協作、多機部署、負載均衡等問題，這正是Kubernetes的主要出發點。
</details>

---

## 第二部分：是非題（5題，每題4分）

**Q16.** Docker是唯一存在的容器運行時（Container Runtime）。 _(True/False)_

<details>
<summary>答案</summary>
<b>False</b> - Docker只不過是眾多容器運行時中最出名的一款而已。還有其他容器運行時實現，如containerd、CRI-O等。
</details>

---

**Q17.** 鏡像的每一層都可以共享和復用，節約存儲和網絡傳輸成本。 _(True/False)_

<details>
<summary>答案</summary>
<b>True</b> - 鏡像內部由多個層組成，這種細粒度結構的好處是相同的層可以共享、復用，節約磁盤存儲和網絡傳輸的成本，也让構建鏡像的工作變得更加容易。
</details>

---

**Q18.** docker命令直接與容器通信，不需要後台服務。 _(True/False)_

<details>
<summary>答案</summary>
<b>False</b> - docker命令只是一個前端工具，它必須與後台服務Docker daemon通信才能實現各種功能。這種架構分離了客戶端和服務端。
</details>

---

**Q19.** 在WordPress示例中，外界可以直接訪問WordPress容器的80端口。 _(True/False)_

<details>
<summary>答案</summary>
<b>False</b> - WordPress容器在啟動時並沒有使用-p參數映射端口號，所以外界是不能直接訪問的。需要通過Nginx反向代理來转发請求。
</details>

---

**Q20.** Docker Registry是最完善的私有鏡像倉庫解决方案。 _(True/False)_

<details>
<summary>答案</summary>
<b>False</b> - Docker Registry是最簡單的私有鏡像倉庫解决方案，功能更完善的還有CNCF Harbor等其他方案。Harbor在後續學習Kubernetes時會介紹。
</details>

---

## 第三部分：填空題（5題，每題4分）

**Q21.** 容器技術彻底变革了應用的**\_\_\_\_**、**\_\_\_\_**與**\_\_\_\_**方式，是「云原生」的根本。

<details>
<summary>答案</summary>
<b>開發、交付、部署</b> 容器技术是后端应用领域的一项重大创新，它彻底变革了应用的開發、交付與部署方式。
</details>

---

**Q22.** 鏡像是容器的**\_\_\_\_**形式，它把應用程序连同依赖的**\_\_\_\_**、配置文件、**\_\_\_\_**等等都打包到了一起。

<details>
<summary>答案</summary>
<b>靜態、操作系統、環境變量</b> 鏡像是容器的靜態形式，它把应用程序连同依赖的操作系統、配置文件、環境變量等打包在一起，能在任何系統上運行。
</details>

---

**Q23.** 操作容器的常用命令有**\_\_\_\_**、**\_\_\_\_**、docker exec、**\_\_\_\_**等。

<details>
<summary>答案</summary>
<b>docker ps、docker run、docker stop</b> 操作容器的常用命令包括docker ps（查看容器）、docker run（運行容器）、docker exec（在容器中執行命令）、docker stop（停止容器）等。
</details>

---

**Q24.** 要給鏡像打標籤並推送到私有倉庫，需要使用**\_\_\_\_**命令打標籤，然後用**\_\_\_\_**命令推送。

<details>
<summary>答案</summary>
<b>docker tag、docker push</b> 使用docker tag命令給鏡像打標籤（加上倉庫地址），然後用docker push命令將鏡像推送到私有倉庫。
</details>

---

**Q25.** Docker的bridge網絡模式中，宿主機固定的IP地址是**\_\_\_\_**，容器IP地址從**\_\_\_\_**開始順序分配。

<details>
<summary>答案</summary>
<b>172.17.0.1、172.17.0.2</b> Docker的bridge網絡模式默認網段是172.17.0.0/16，宿主機固定是172.17.0.1，容器IP從172.17.0.2開始順序分配（如果之前沒有其他容器）。
</details>

---

## 第四部分：簡答題（3題，每題10分）

**Q26.** 說明鏡像的層（Layer）結構及其優點。

<details>
<summary>答案</summary>
鏡像內部由多個層組成，每一層都是一組文件，多個層使用Union FS技術合併成一個文件系統供容器使用。

<b>主要優點：</b>
1. 相同的層可以共享、復用，節約磁盤存儲空間
2. 減少網絡傳輸成本，推送/拉取鏡像時只傳輸變化的層
3. 让構建鏡像的工作變得更加容易，可以基於現有鏡像層構建新鏡像
4. 提高了鏡像的分發效率和管理便利性
</details>

---

**Q27.** 解釋為什麼在WordPress容器中需要設置WORDPRESS_DB_HOST為MariaDB的IP地址，而不是使用localhost。

<details>
<summary>答案</summary>
<b>原因分析：</b>

1. <b>容器隔離：</b> 每個容器都有自己独立的網絡命名空間，localhost在容器內指向的是容器自己，而不是宿主機或其他容器。

2. <b>網絡通信：</b> WordPress和MariaDB運行在不同的容器中，需要通過Docker的bridge網絡進行通信。MariaDB有自己的容器IP地址（如172.17.0.2）。

3. <b>端口隔离：</b> MariaDB的3306端口被容器隔离，外界不可見，WordPress只能通過容器間的bridge網絡訪問MariaDB。

4. <b>正確配置：</b> 必須使用MariaDB容器的實際IP地址（如172.17.0.2）才能建立正確的網絡連接，使用localhost會導致連接失敗。

<b>解決方案：</b> 使用docker inspect命令查看MariaDB容器的IP地址，然後將WORDPRESS_DB_HOST設置為該地址。在Kubernetes中會使用Service對象來解決這個問題。
</details>

---

**Q28.** 根據課程內容，容器技術存在哪些不足之处？容器编排技術需要解決哪些問題？

<details>
<summary>答案</summary>
<b>容器技術的不足之处：</b>

1. <b>手動操作：</b> 要手動運行命令啟動應用，人工確認運行狀態
2. <b>多容器協作困難：</b> 運行多個容器組成的應用比較麻煩，需要人工干预（如檢查IP地址）才能維護網絡通信
3. <b>單機限制：</b> 現有的網絡模式功能只適合單機，多台服務器上運行應用、負載均衡缺乏解決方案
4. <b>擴展困難：</b> 要增加應用數量時，容器技術本身無法自動處理

<b>容器編排需要解決的問題：</b>

1. <b>自動化管理：</b> 容器的自動啟動、停止、監控和維護
2. <b>網絡配置：</b> 容器間的網絡通信配置自動化，不需要手動查看IP地址
3. <b>多機部署：</b> 在多台服務器上部署應用，實現分布式運行
4. <b>負載均衡：</b> 自動創建和管理負載均衡，合理分配流量
5. <b>水平擴展：</b> 根據需求自動增加或減少容器數量
6. <b>數據持久化：</b> 管理容器的數據存儲和持久化
7. <b>故障恢復：</b> 容器失敗時自動重啟或替換
8. <b>配置管理：</b> 集中管理應用配置和環境變量

<b>實現方式：</b> 將docker run命令整理成腳本，加上Shell、Python編程實現自動化，形成容器編排的雏形。Kubernetes正是為解決這些問題而設計的容器編排平台。
</details>

---

## 第五部分：配對題（2題，每題5分）

**Q29.** 將下列Docker命令與其功能進行配對：

| 命令             | 功能                      |
| ---------------- | ------------------------- |
| 1. docker ps     | A. 拉取鏡像                |
| 2. docker pull   | B. 推送鏡像到倉庫          |
| 3. docker push   | C. 查看運行中的容器        |
| 4. docker tag    | D. 刪除鏡像                |
| 5. docker rmi    | E. 给鏡像打標籤            |

<details>
<summary>答案</summary>
1-C, 2-A, 3-B, 4-E, 5-D

<b>說明：</b>
- docker ps：查看當前運行中的容器列表
- docker pull：從鏡像倉庫拉取鏡像到本地
- docker push：將本地鏡像推送到鏡像倉庫
- docker tag：給鏡像打標籤，通常用於標記倉庫地址
- docker rmi：删除本地鏡像
</details>

---

**Q30.** 將下列WordPress架構中的组件與其角色進行配對：

| 组件      | 角色                          |
| --------- | ----------------------------- |
| 1. Nginx  | A. 數據庫服務器，端口3306     |
| 2. MariaDB| B. 反向代理，對外暴露80端口   |
| 3. WordPress | C. 应用服務器，使用MariaDB存儲數據 |

<details>
<summary>答案</summary>
1-B, 2-A, 3-C

<b>說明：</b>
- <b>Nginx：</b> 前端反向代理，對外暴露80端口，將請求转发給WordPress
- <b>MariaDB：</b> 后端關系型數據庫，端口3306，但被容器隔离不對外暴露
- <b>WordPress：</b> 中間应用服務器，使用MariaDB存儲數據，自身80端口不對外暴露，通過Nginx代理访问
</details>

---

## 答案總覽

### 第一部分：選擇題
1. A | 2. B | 3. C | 4. B | 5. B
6. C | 7. C | 8. B | 9. C | 10. C
11. C | 12. C | 13. B | 14. B | 15. B

### 第二部分：是非題
16. False | 17. True | 18. False | 19. False | 20. False

### 第三部分：填空題
21. 開發、交付、部署
22. 靜態、操作系統、環境變量
23. docker ps、docker run、docker stop
24. docker tag、docker push
25. 172.17.0.1、172.17.0.2

### 第四部分：簡答題
26. 參見詳細答案
27. 參見詳細答案
28. 參見詳細答案

### 第五部分：配對題
29. 1-C, 2-A, 3-B, 4-E, 5-D
30. 1-B, 2-A, 3-C

---

_生成來源: 07｜实战演练：玩转Docker - Kubernetes入门实战课_
_測驗難度分布: 60%基礎, 30%中等, 10%進階_
_題型分布: 15選擇題, 5是非題, 5填空題, 3簡答題, 2配對題_