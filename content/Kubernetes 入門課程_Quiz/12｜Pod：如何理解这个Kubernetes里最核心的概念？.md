# Pod：Kubernetes 最核心概念測驗

**科目：** Kubernetes 入門課程
**主題：** Pod 核心概念
**難度：** 混合（基礎、中等、進階）
**題數：** 15題
**建議時間：** 25分鐘

---

## 第一部分：選擇題（每題5分，共30分）

**Q1.** Pod 這個詞的原意是什麼？為什麼 Kubernetes 使用這個名稱？

A) Pod 原意是「豆子」，代表單一的容器
B) Pod 原意是「豌豆荚」，象徵包含多個组件的結構
C) Pod 原意是「太空艙」，代表獨立的運行環境
D) Pod 原意是「艦艇」，代表大型應用系統

<details>
<summary>答案</summary>
<b>B) Pod 原意是「豌豆荚」，象徵包含多個组件的結構</b>
Pod 原意是豌豆荚，延伸出「舱室」「太空舱」等含义。形象地说，Pod 就是包含了很多组件、成员的一种结构，就像豌豆荚里包含多颗豌豆一样，Pod 里包含多个容器。
</details>

---

**Q2.** 在 Kubernetes 中，為什麼不能把多個密切協作的應用都放在一個容器裡運行？

A) 因為 Kubernetes 不支持多進程容器
B) 因為這會增加容器的體積，消耗更多資源
C) 因為這違背了容器的理念，容器應該對單一應用進行獨立封裝
D) 因為多應用容器會導致網絡配置複雜

<details>
<summary>答案</summary>
<b>C) 因為這違背了容器的理念，容器應該對單一應用進行獨立封裝</b>

容器的理念是对应用的独立封装，它里面就应该是一个进程、一个应用。如果里面有多个应用，不仅违背了容器的初衷，也会让容器更难以管理。
</details>

---

**Q3.** 在 Pod 的 YAML 文件中，spec.containers 字段是什麼類型？

A) 單一容器對象
B) 字典（map）類型
C) 数组（list）類型，可以包含多個容器
D) 字符串類型，指定容器名稱

<details>
<summary>答案</summary>
<b>C) 数组（list）類型，可以包含多個容器</b>

spec.containers 是一个数组，里面的每一个元素又是一个 container 对象，也就是容器。这允许 Pod 包含多个容器，它们作为一个整体被调度和运行。
</details>

---

**Q4.** imagePullPolicy 的默認值是什麼（當使用非 latest 標籤時）？

A) Always
B) Never
C) IfNotPresent
D) Default

<details>
<summary>答案</summary>
<b>C) IfNotPresent</b>

imagePullPolicy 默认是 IfNotPresent，也就是说只有本地不存在才会远程拉取镜像，可以减少网络消耗。但如果使用 latest 标签，默认值会是 Always。
</details>

---

**Q5.** 在 Pod 的 YAML 中，command 和 args 字段分別對應 Dockerfile 中的什麼指令？

A) command 對應 CMD，args 對應 ENTRYPOINT
B) command 對應 ENTRYPOINT，args 對應 CMD
C) command 對應 RUN，args 對應 CMD
D) command 對應 CMD，args 對應 RUN

<details>
<summary>答案</summary>
<b>B) command 對應 ENTRYPOINT，args 對應 CMD</b>

command 定义容器启动时要执行的命令，相当于 Dockerfile 里的 ENTRYPOINT 指令。args 是 command 运行时的参数，相当于 Dockerfile 里的 CMD 指令。这与 Docker 的含义不同，要特别注意。
</details>

---

**Q6.** 當 Pod 出現 CrashLoopBackOff 狀態時，應該使用哪個 kubectl 命令來查看詳細的調試信息？

A) kubectl get pod
B) kubectl logs
C) kubectl describe pod
D) kubectl exec

<details>
<summary>答案</summary>
<b>C) kubectl describe pod</b>

kubectl describe 命令可以检查 Pod 的详细状态，在调试排错时很有用。通常需要关注末尾的 "Events" 部分，它显示的是 Pod 运行过程中的一些关键节点事件。
</details>

---

## 第二部分：是非題（每題4分，共16分）

**Q7.** Pod 是 Kubernetes 世界裡的「原子」，內部是完全統一的整體，沒有任何結構。 _(是/否)_

<details>
<summary>答案</summary>
<b>否</b> - Pod 虽然被称为 Kubernetes 世界里的「原子」，但这个「原子」内部是有结构的（包含多个容器），不是铁板一块。Pod 打包了多个容器，让它们保持相对独立又能够小范围共享网络、存储等资源。
</details>

---

**Q8.** 在 Kubernetes 中，Pod 必須要有名字，這是所有資源對象的共同約定。 _(是/否)_

<details>
<summary>答案</summary>
<b>是</b> - 在 Kubernetes 里，Pod 必须要有一个名字，这也是 Kubernetes 里所有资源对象的一个约定。与 Docker 不同，Docker 创建容器时可以不给容器起名字。
</details>

---

**Q9.** Pod 中的容器總是會被一起調度、一起運行，絕不會出現分離的情況。 _(是/否)_

<details>
<summary>答案</summary>
<b>是</b> - Pod 是对容器的「打包」，里面的容器是一个整体，总是能够一起调度、一起运行，绝不会出现分离的情况。这正是 Pod 存在的意义之一。
</details>

---

**Q10.** 在實際生產環境中，通常會直接創建 Pod 來部署應用。 _(是/否)_

<details>
<summary>答案</summary>
<b>否</b> - 虽然 Pod 是 Kubernetes 的核心概念，非常重要，但事实上在 Kubernetes 里通常并不会直接创建 Pod。因为 Pod 只是对容器做了简单的包装，比较脆弱，离复杂的业务需求还有些距离，需要 Job、CronJob、Deployment 等其他对象增添更多的功能才能投入生产使用。
</details>

---

## 第三部分：填空題（每題6分，共18分）

**Q11.** Pod 是 Kubernetes 管理應用的最小单位，其他的所有概念都是从 Pod **\_\_\_\_\_\_** 出来的。

<details>
<summary>答案</summary>
<b>衍生</b>

Pod 是 Kubernetes 管理应用的最小单位，其他的所有概念都是从 Pod 衍生出来的。所有的 Kubernetes 资源都直接或者间接地依附在 Pod 之上，所有的 Kubernetes 功能都必须通过 Pod 来实现。
</details>

---

**Q12.** labels 字段可以添加任意数量的 **\_\_\_\_\_\_** 给 Pod「贴」上归类的标签，结合 name 就更方便识别和管理了。

<details>
<summary>答案</summary>
<b>Key-Value</b>

labels 字段可以添加任意数量的 Key-Value，给 Pod「贴」上归类的标签。例如可以使用标签 env=dev/test/prod，或者 region=north/south，tier=front/middle/back 等。
</details>

---

**Q13.** 在 YAML 文件中，Pod 的 apiVersion 字段值是 **\_\_\_\_\_\_**，kind 字段值是 Pod。

<details>
<summary>答案</summary>
<b>v1</b>

对于 Pod 来说，apiVersion 和 kind 这两个字段分别是固定的值 v1 和 Pod。这是 Pod 作为 API 对象的基本组成部分。
</details>

---

## 第四部分：簡答題（每題8分，共16分）

**Q14.** 請說明為什麼需要 Pod 這個概念？它解決了什麼問題？

<details>
<summary>答案</summary>
Pod 解決了現實生產環境中多個應用密切協作的問題。

**關鍵點包括：**

- **容器協作需求**：现实中经常会有多个进程密切协作才能完成任务的应用，例如 WordPress 网站需要 Nginx、WordPress、MariaDB 三个容器一起工作
- **紧密耦合的應用**：有些应用结合得非常紧密以至于无法拆开，如需要其他应用初始化配置、日志代理需要读取另一个应用的本地磁盘文件
- **保持容器理念**：不能把多应用放在一个容器里，因为违背了容器的独立封装理念
- **提供「收納艙」**：Pod 让多个容器既保持相对独立，又能够小范围共享网络、存储等资源，而且永远是「绑在一起」的状态
- **整體調度**：Pod 里的容器作为一个整体被调度、运行，绝不会分离
</details>

---

**Q15.** Pod 和容器之間有什麼區別和聯繫？請從概念、粒度和功能角度進行分析。

<details>
<summary>答案</summary>
Pod 和容器既有區別又有聯繫，Pod 是更高層次的抽象。

**關鍵點包括：**

- **概念層面**：容器是对进程的独立封装，Pod 是对多个容器的「打包」和抽象
- **粒度角度**：容器是「细粒度」，虚拟机是「粗粒度」，Pod 是「中粒度」，灵活又轻便
- **功能差異**：容器提供隔离性，Pod 在保持隔离的同时提供小范围共享（网络、存储等）
- **管理層級**：Kubernetes 通过 Pod 编排容器，Pod 是调度的最小单位，容器不是直接调度对象
- **關係比喻**：Pod 和容器的关系就像豌豆荚和豌豆、客厅和预制房间、进程组和进程
- **依賴性**：Pod 属于 Kubernetes，可以在不触碰下层容器的情况下任意定制修改
</details>

---

## 第五部分：配對題（10分）

**Q16.** 請將 kubectl 命令與其功能描述進行配對：

| 命令 | 功能描述 |
| ----- | -------- |
| 1. kubectl apply -f | A. 查看 Pod 的标准输出流信息（日志） |
| 2. kubectl logs | B. 进入 Pod 内部执行 Shell 命令 |
| 3. kubectl describe pod | C. 使用 YAML 文件创建或更新 Pod |
| 4. kubectl exec -it | D. 查看 Pod 的详细状态和 Events 事件 |
| 5. kubectl cp | E. 将本地文件拷贝进 Pod |

<details>
<summary>答案</summary>
1-C, 2-A, 3-D, 4-B, 5-E

<b>詳細說明：</b>
- **kubectl apply -f**：使用 YAML 文件创建资源对象，是声明式管理方式
- **kubectl logs**：相当于 Docker 的 logs，展示容器运行日志
- **kubectl describe pod**：在调试排错时很有用，关注末尾的 "Events" 部分
- **kubectl exec -it**：需要加 -- 分隔符，如 kubectl exec -it ngx-pod -- sh
- **kubectl cp**：与 Docker 的 cp 命令类似，可双向拷贝文件
</details>

---

## 總結

本測驗涵蓋了 Pod 的核心概念、YAML 描述方法、kubectl 操作命令等關鍵知識點。Pod 作為 Kubernetes 的「原子」單元，是理解 Kubernetes 架構的基礎。掌握 Pod 的概念和使用方法，是 Kubernetes 學習之旅成功的一半。

---

**生成來源：** 《Kubernetes入門實戰課》第12講 - Pod：如何理解这个Kubernetes里最核心的概念？
**作者：** Chrono
**日期：** 2022-07-18