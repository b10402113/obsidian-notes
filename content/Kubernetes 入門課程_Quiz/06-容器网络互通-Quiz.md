# Kubernetes 容器网络互通 Quiz

**課程:** Kubernetes 入門實戰課
**主題:** 06｜打破次元壁：容器該如何與外界互聯互通
**難度:** 混合（基礎 40%，中等 40%，進階 20%）
**題數:** 15題
**建議時間:** 25-30分鐘

---

## Section A: 單選題 (MCQ) - 30分

**Q1.** 在 Docker 中，要在宿主機和容器之間拷貝文件，應該使用哪個命令？

A) docker copy
B) docker cp
C) docker transfer
D) docker move

<details>
<summary>答案</summary>
<b>B) docker cp</b>

docker cp 命令是 Docker 提供的基本數據交換功能，可以在宿主機和容器之間互相拷貝文件。用法類似 Linux 的 cp 和 scp 命令，指定源路徑和目標路徑即可。如果源路徑是宿主機則將文件拷貝進容器，反之則從容器拷貝出到宿主機。
</details>

---

**Q2.** 使用 docker run 命令啟動容器時，要讓容器共享宿主機的某個目錄，應該使用哪個參數？

A) -c
B) -d
C) -v
D) -p

<details>
<summary>答案</summary>
<b>C) -v</b>

-v 參數用于掛載宿主機目錄到容器內，格式為「宿主機路徑:容器內路徑」。這樣容器和宿主機就能共享同一個目錄，兩邊對文件的操作會即時同步，不需要反覆拷貝，效率更高。
</details>

---

**Q3.** Docker 提供的三種網絡模式中，哪一種模式讓容器直接使用宿主機的網絡？

A) null
B) host
C) bridge
D) overlay

<details>
<summary>答案</summary>
<b>B) host</b>

host 模式讓容器直接使用宿主機網絡，相當于去掉了容器的網絡隔離。所有容器共享宿主機的 IP 地址和網卡，通信效率高但容易導致端口衝突。使用時需要在 docker run 加上 --net=host 參數。
</details>

---

**Q4.** Docker 的默認網絡模式是什麼？

A) null
B) host
C) bridge
D) none

<details>
<summary>答案</summary>
<b>C) bridge</b>

bridge 模式是 Docker 的默認網絡模式。它通過軟件虛擬出網橋（docker0），容器和宿主機通過虛擬網卡接入這個網橋，在私有網段內互相通信。一般不需要顯式指定 --net=bridge。
</details>

---

**Q5.** 在 bridge 網絡模式下，要將宿主機的 8080 端口映射到容器的 80 端口，應該如何使用 docker run 命令？

A) docker run -p 80:8080
B) docker run -p 8080:80
C) docker run -v 8080:80
D) docker run --port 8080:80

<details>
<summary>答案</summary>
<b>B) docker run -p 8080:80</b>

-p 參數用于端口映射，格式為「宿主機端口:容器端口」。docker run -p 8080:80 表示將宿主機的 8080 端口映射到容器內部的 80 端口。這樣可以避免多個容器使用相同端口時的衝突問題。
</details>

---

**Q6.** docker cp 命令與 Dockerfile 中的 COPY 指令的主要區別是什麼？

A) docker cp 可以拷貝更多文件類型
B) COPY 指令是在構建鏡像時拷貝文件到鏡像層，docker cp 是在容器運行時拷貝文件到容器
C) docker cp 更快，COPY 指令更慢
D) 它們功能完全相同，只是命令不同

<details>
<summary>答案</summary>
<b>B) COPY 指令是在構建鏡像時拷貝文件到鏡像層，docker cp 是在容器運行時拷貝文件到容器</b>

COPY 指令是在構建鏡像階段執行的，文件會被永久打包進鏡像的只讀層，存在於鏡像中。docker cp 命令是在容器運行階段執行的，將文件拷貝到容器的可寫層，是臨時性的操作。COPY 需要在「構建上下文」路徑內的文件，而 docker cp 可以拷貝宿主機任意位置的文件。
</details>

---

## Section B: 是非題 (True/False) - 20分

**Q7.** 使用 -v 參數掛載宿主機目錄時，容器內對文件的操作會即時同步到宿主機。 _(True/False)_

<details>
<summary>答案</summary>
<b>True</b>

-v 參數讓容器共享宿主機的目錄，兩邊看到的是同一份文件。在容器內對目錄下文件的操作（如刪除、新建、修改）會即時反映到宿主機，反之亦然。這種共享方式避免了數據拷貝，效率更高，適合開發測試場景。
</details>

---

**Q8.** host 網絡模式具有完全的網絡隔離性，容器使用獨立的 IP 地址。 _(True/False)_

<details>
<summary>答案</summary>
<b>False</b>

host 網絡模式相當于去掉了容器的網絡隔離，容器直接使用宿主機的 IP 地址和網卡，沒有獨立的 IP。這種模式通信效率高，但缺少網絡隔離，多個容器運行時容易導致端口衝突。bridge 模式才提供網絡隔離，容器有獨立的私有 IP 地址。
</details>

---

**Q9.** 在 bridge 模式下，容器的 IP 地址通常是私有地址，例如 172.17.0.x 網段。 _(True/False)_

<details>
<summary>答案</summary>
<b>True</b>

bridge 模式下，Docker 會創建虛擬網橋 docker0，默認網段是 172.17.0.0/16。容器啟動後會分配該網段內的私有 IP 地址，如 172.17.0.2、172.17.0.3 等，宿主機則使用 172.17.0.1。容器間可以通過這些 IP 地址互相通信。
</details>

---

**Q10.** 使用 docker exec 命令可以進入正在運行的容器內執行命令或操作。 _(True/False)_

<details>
<summary>答案</summary>
<b>True</b>

docker exec 命令用于在已經運行的容器內執行新命令。常用形式是 docker exec -it <容器ID> sh，這樣可以進入容器的交互式 shell 環境，查看文件、執行命令、檢查網絡配置等。課程中多次使用此命令來驗證文件拷貝和查看網卡信息。
</details>

---

## Section C: 填空題 (Fill-in-the-Blank) - 20分

**Q11.** Docker 提供的三種網絡模式分别是 **______**、**______** 和 **______**。

<details>
<summary>答案</summary>
<b>null, host, bridge</b>

Docker 提供三種網絡模式：
- null：無網絡模式，允許自定義網絡插件
- host：容器直接使用宿主機網絡，無網絡隔離
- bridge：虛擬網橋模式，容器有獨立私有 IP，是默認網絡模式
</details>

---

**Q12.** 要將當前目錄下的 a.txt 文件拷貝到容器 ID 為 062 的容器的 /tmp 目錄，命令是：docker cp a.txt **______**:/tmp

<details>
<summary>答案</summary>
<b>062</b>

docker cp 命令格式為 docker cp <源路徑> <目標路徑>。目標路徑如果是容器內路徑，需要用容器名或容器 ID 指明，格式為「容器ID:容器內路徑」或「容器名:容器內路徑」。
</details>

---

**Q13.** 使用 host 網絡模式啟動容器時，需要在 docker run 命令中添加 **______** 參數。

<details>
<summary>答案</summary>
<b>--net=host</b>

要使用 host 網絡模式，需要在 docker run 命令中添加 --net=host 參數。例如：docker run -d --rm --net=host nginx:alpine。這樣容器就會直接使用宿主機的網絡棧，與宿主機共享 IP 地址和網卡。
</details>

---

**Q14.** 在 bridge 網絡模式下，容器間可以通過 **______** 地址實現網絡通信，也可以使用 docker inspect 命令查看容器 IP。

<details>
<summary>答案</summary>
<b>IP</b>

在 bridge 模式下，容器會分配私有網段的 IP 地址（如 172.17.0.x），容器之間可以通過這些 IP 地址互相通信。也可以使用 docker inspect <容器ID> |grep IPAddress 命令查看容器的具體 IP 地址。
</details>

---

## Section D: 簡答題 (Short Answer) - 20分

**Q15.** 請說明 docker cp 命令與 Dockerfile 中 COPY 指令的三個主要區別。

<details>
<summary>答案</summary>
<b>docker cp 與 COPY 指令的主要區別：</b>

**1. 執行階段不同：**
- COPY 指令在構建鏡像階段執行，文件被永久打包進鏡像的只讀層
- docker cp 在容器運行階段執行，文件拷貝到容器的可寫層，是臨時性操作

**2. 文件持久性不同：**
- COPY 指令拷貝的文件會永遠存在于鏡像中，任何使用該鏡像啟動的容器都有這些文件
- docker cp 拷貝的文件只存在于當前容器，容器刪除後文件就消失

**3. 路徑限制不同：**
- COPY 指令只能拷貝「構建上下文」路徑內的文件
- docker cp 可以拷貝宿主機任意位置的文件到容器，或從容器拷貝到宿主機任意位置

**補充要點：**
- COPY 生成新的鏡像層，docker cp 不生成新層
- COPY 不受 namespace 約束，docker cp 在容器運行時操作
</details>

---

**Q16.** 請分析 host 網絡模式與 bridge 網絡模式的優缺點及適用場景。

<details>
<summary>答案</summary>
<b>host 模式與 bridge 模式的優缺點分析：</b>

**Host 網絡模式：**

優點：
- 通信效率高，無中間層（虛擬網橋和網卡），直接使用宿主機網絡
- 网络配置简单，无需额外设置
- 适合对网络性能要求高的场景

缺點：
- 缺少网络隔离，容器使用宿主机的 IP 和网卡
- 容易导致端口冲突，不适合运行大量容器
- 安全性较低，容器可直接访问宿主机网络

適用場景：
- 小规模集群，容器数量少
- 集群边界需要与外界通信的场景，如 ingress-nginx
- 性能敏感的网络应用
- 开发测试环境，端口不易冲突

**Bridge 網絡模式：**

優點：
- 提供网络隔离，容器有独立私有 IP
- 端口管理灵活，通过端口映射避免冲突
- 安全性较好，容器间网络相互隔离
- 适合大规模部署，可配合网络插件扩展功能（如 VXLAN、流量控制等）
- Kubernetes 默认采用类似架构

缺點：
- 通信效率略低，多了虚拟网桥和网卡的中转
- 网络配置相对复杂
- 需要理解虚拟网络概念

適用場景：
- 大规模集群，容器数量多
- 生产环境，需要网络隔离和灵活端口管理
- Kubernetes 集群内部通信
- 需要配合 CNI 插件（如 flannel、Calico）的场景

**關鍵比較點：**
- host 效率优先，bridge 隔离优先
- host 适合小规模边界场景，bridge 适合大规模内部场景
- 实践中 bridge 模式使用更广泛
</details>

---

**Q17.** 說明 -v 參數掛載目錄在實際開發工作中的應用場景和優勢。

<details>
<summary>答案</summary>
<b>-v 參數掛載目錄的應用場景與優勢：</b>

**主要優勢：**

1. **即時同步**：容器和宿主機共享同一目錄，修改即時生效，無需反覆拷貝
2. **效率提升**：避免數據拷貝操作，減少時間和空間占用
3. **靈活性高**：可以在不變動本機環境前提下使用容器運行不同版本應用

**典型應用場景：**

1. **多版本開發環境：**
   - 本機有 Python 2.7，需用 Python 3 开发
   - 拉取 Python 3 镜像，挂载本地代码目录
   - 在容器内使用 Python 3 运行和测试代码
   - 不影响本机 Python 2.7 环境

2. **配置文件共享：**
   - 将本地配置文件挂载到容器配置目录
   - 如挂载 nginx.conf 到 /etc/nginx/nginx.conf
   - 修改配置文件后容器即时生效

3. **源码开发测试：**
   - 本地编写代码，挂载到容器
   - 在容器内安装依赖包并运行
   - 适合频繁修改的开发测试工作

4. **数据库数据持久化：**
   - 挂载宿主机目录存储数据库文件
   - 容器删除后数据不丢失

**使用示例：**
```bash
# 挂载当前目录到容器 /tmp 目录
docker run -it --rm -v `pwd`:/tmp python:alpine sh

# 挂载宿主机 /tmp 到容器 /tmp
docker run -d --rm -v /tmp:/tmp redis
```

**關鍵要點：**
- 相比 docker cp，-v 更适合频繁修改的开发场景
- 相比打包到镜像，-v 更灵活，无需重新构建镜像
- 实现了「有限的隔离」，既保持隔离优势又打通数据通道
</details>

---

## Section E: 配對題 (Matching) - 10分

**Q18.** 請將以下 Docker 網絡模式與其特點進行配對：

| 模式        | 特點描述                          |
| ----------- | --------------------------------- |
| 1. null     | A. 容器與宿主機共享 IP 和網卡      |
| 2. host     | B. 虛擬網橋，容器有獨立私有 IP      |
| 3. bridge   | C. 無網絡，允許自定義網絡插件      |

<details>
<summary>答案</summary>
<b>1-C, 2-A, 3-B</b>

配對說明：
- null 模式：不提供網絡功能，但允許其他網絡插件自定義網絡連接
- host 模式：容器直接使用宿主機網絡，共享 IP 地址和網卡，通信效率高但缺少隔離
- bridge 模式：軟件虛擬網橋模式，容器和宿主機通過虛擬網卡接入網橋，有獨立私有 IP，是 Docker 默認網絡模式
</details>

---

**Q19.** 請將以下 Docker 命令/參數與其功能進行配對：

| 命令/參數    | 功能描述                          |
| ----------- | --------------------------------- |
| 1. docker cp| A. 端口映射，格式為「宿主機端口:容器端口」|
| 2. -v       | B. 在宿主機和容器之間拷貝文件      |
| 3. -p       | C. 挂載宿主机目录到容器，格式為「宿主机路径:容器路径」|

<details>
<summary>答案</summary>
<b>1-B, 2-C, 3-A</b>

配對說明：
- docker cp：基本數據交換命令，可双向拷贝文件
- -v 参数：目录挂载参数，实现宿主机与容器目录共享，适合频繁文件交互场景
- -p 参数：端口映射参数，将宿主机端口映射到容器端口，避免端口冲突
</details>

---

## 總結與建議

本 Quiz 涵蓋了 Docker 容器與外部系統互聯互通的核心概念：

### 重點知識回顧

1. **數據交換**：docker cp 和 -v 挂载目录两种方式各有优劣
2. **網絡模式**：host 和 bridge 模式的選擇需根據場景決定
3. **端口管理**：-p 端口映射解决了多容器端口冲突问题
4. **實踐應用**：理解如何在实际开发中应用这些技术

### 學習建議

- 熟練掌握 docker cp 和 -v 的使用場景區別
- 理解 host 和 bridge 模式的底層原理
- 嘗試在實際項目中應用掛載目錄和端口映射
- 思考如何將 Redis、MySQL、Node.js 等服務容器化並與外界通信

---

_Generated from: PDF/Kubernetes 入門課程/06｜打破次元壁：容器该如何与外界互联互通.pdf_
_Generated date: 2026-05-11_