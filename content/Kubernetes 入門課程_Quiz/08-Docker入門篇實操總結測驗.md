# Docker 入門篇實操總結測驗

**科目:** Kubernetes 入門實戰課
**難度:** 混合 (簡單、中等、困難)
**題數:** 20 題
**建議時間:** 25-30 分鐘

---

## 第一部分：選擇題 (每題 3 分，共 30 分)

**Q1.** 在 Docker 中，查看當前系統的 CPU、內存、容器數量、鏡像數量等資訊，應該使用哪個命令？

A) docker version
B) docker info
C) docker ps
D) docker images

<details>
<summary>答案</summary>
<b>B) docker info</b>

<b>解析：</b>docker info 顯示當前系統相關的詳細資訊，包括 CPU、內存、容器數量、鏡像數量、容器運行時、存儲文件系統等。docker version 只顯示 Docker Engine 版本資訊；docker ps 顯示容器列表；docker images 顯示鏡像列表。
</details>

---

**Q2.** 關於容器的特性，以下哪項描述是正確的？

A) 容器與宿主機完全隔離，包括內核和時間
B) 容器就是操作系統裡的進程，加上 namespace、cgroup、chroot 的限制
C) 容器啟動速度比虛擬機慢，資源利用率較低
D) 容器運行時需要完整的操作系統

<details>
<summary>答案</summary>
<b>B) 容器就是操作系統裡的進程，加上 namespace、cgroup、chroot 的限制</b>

<b>解析：</b>容器其實是操作系統裡的進程，只是被容器運行環境加上了 namespace、cgroup、chroot 的限制。容器沒有隔離的部分包括時間和內核（與宿主機共享）。因為沒有虛擬機的成本，容器啟動更迅速，資源利用率更高。
</details>

---

**Q3.** 在 Dockerfile 中，ARG 指令與 ENV 指令的主要區別是什麼？

A) ARG 定義的變量在構建和運行時都可以使用
B) ENV 定義的變量只能在構建時使用
C) ARG 定義的變量只能在構建鏡像時使用，ENV 定義的變量會在容器運行時以環境變量形式出現
D) 兩者功能完全相同，可以互換使用

<details>
<summary>答案</summary>
<b>C) ARG 定義的變量只能在構建鏡像時使用，ENV 定義的變量會在容器運行時以環境變量形式出現</b>

<b>解析：</b>ARG 指令定義的變量只能在構建鏡像的時候使用，而 ENV 定義的變量則會在容器運行的時候以環境變量的形式出現，讓進程運行時使用。
</details>

---

**Q4.** 關於 Dockerfile 中的 COPY 指令，以下哪項描述是錯誤的？

A) COPY 指令會把構建上下文裡的文件拷貝到鏡像中
B) COPY 指令可以使用絕對路徑
C) COPY 指令必須使用構建上下文的相對路徑
D) Docker 會把構建上下文裡的所有文件打包傳遞給 docker daemon

<details>
<summary>答案</summary>
<b>B) COPY 指令可以使用絕對路徑</b>

<b>解析：</b>COPY 指令不能使用絕對路徑，必須是構建上下文的相對路徑。Docker 會把構建上下文裡的所有文件打包傳遞給 docker daemon，所以應該盡量只包含必要的文件。
</details>

**Q5.** 使用 docker run 命令時，想要在容器啟動後進入交互式 shell，應該添加哪些參數？

A) -d
B) -it
C) --rm
D) -p

<details>
<summary>答案</summary>
<b>B) -it</b>

<b>解析：</b>-it 參數組合表示交互式終端（interactive + tty），讓用戶可以進入容器內部的 shell 環境。-d 表示後台運行；--rm 表示容器停止後自動刪除；-p 用於端口映射。
</details>

---

**Q6.** 在 Docker 中，將本機的 /tmp 目錄掛載到容器的 /tmp 目錄，應該使用哪個參數？

A) docker cp
B) -v /tmp:/tmp
C) -p 80:80
D) docker exec

<details>
<summary>答案</summary>
<b>B) -v /tmp:/tmp</b>

<b>解析：</b>-v 參數用於掛載本地目錄到容器內部，格式為 -v 宿主機路徑:容器路徑。docker cp 是手動拷貝文件的命令；-p 用於端口映射；docker exec 用於在運行中的容器內執行命令。
</details>

---

**Q7.** 構建 Docker 鏡像時，使用哪個命令可以為鏡像打上標籤？

A) docker save
B) docker load
C) docker build -t
D) docker tag

<details>
<summary>答案</summary>
<b>C) docker build -t</b>

<b>解析：</b>docker build 命令配合 -t 參數可以在構建時為鏡像打上標籤，例如 docker build -t ngx-app:1.0 .。雖然 docker tag 也可以為已有的鏡像打標籤，但在構建過程中使用 -t 更為直接。docker save 和 docker load 用於導出和導入鏡像。
</details>

---

**Q8.** 容器停止後，以下哪個命令可以查看已結束的容器？

A) docker ps
B) docker ps -a
C) docker images
D) docker rm

<details>
<summary>答案</summary>
<b>B) docker ps -a</b>

<b>解析：</b>docker ps 只顯示正在運行的容器，docker ps -a 會顯示所有容器，包括已停止的容器。docker images 顯示鏡像列表；docker rm 用於刪除容器。
</details>

---

**Q9.** 關於 EXPOSE 指令，以下哪項描述是正確的？

A) EXPOSE 指令會自動將容器端口映射到宿主機
B) EXPOSE 指令聲明容器對外服務的端口號
C) EXPOSE 指令與 -p 參數功能相同
D) EXPOSE 指令必須放在 Dockerfile 的最後

<details>
<summary>答案</summary>
<b>B) EXPOSE 指令聲明容器對外服務的端口號</b>

<b>解析：</b>EXPOSE 指令只是聲明容器對外服務的端口號，不會自動映射端口。要實際映射端口，需要使用 docker run 的 -p 參數。EXPOSE 指令可以放在 Dockerfile 的任意位置。
</details>

---

**Q10.** 在容器內部查看網卡情況，發現 eth0 是虛擬網卡，IP 地址是 172.17.0.2，這是什麼類型的地址？

A) A 類私有地址
B) B 類私有地址
C) C 類私有地址
D) 公網地址

<details>
<summary>答案</summary>
<b>B) B 類私有地址</b>

<b>解析：</b>172.17.0.2 屬於 B 類私有地址範圍（172.16.0.0 - 172.31.255.255）。Docker 默認為容器分配 B 類私有地址，這是在容器內部網絡環境中使用的虛擬網卡 IP 地址。
</details>

---

## 第二部分：是非題 (每題 2 分，共 10 分)

**Q11.** 容器與宿主機共享內核，因此在容器內執行 uname -a 與在宿主機上執行的結果完全一致（除了主機名）。_(對/錯)_

<details>
<summary>答案</summary>
<b>對</b>

<b>解析：</b>容器沒有隔離內核，與宿主機共享同一個內核。因此在容器內執行 uname -a 查看內核版本時，結果與宿主機一致，只是主機名（hostname）會顯示容器的 ID。
</details>

---

**Q12.** 使用 docker run -d --rm nginx:alpine 啟動容器後，容器會在停止後自動刪除。_(對/錯)_

<details>
<summary>答案</summary>
<b>對</b>

<b>解析：</b>--rm 參數表示容器在停止後會自動刪除。這在臨時測試或運行一次性任務時非常有用，可以避免累積大量已停止的容器。
</details>

---

**Q13.** docker cp 命令只能在容器運行時使用，無法在容器停止後拷貝文件。_(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b>

<b>解析：</b>docker cp 命令可以在容器停止後仍然使用，只要容器還存在（未刪除），就可以拷貝文件。即使容器處於停止狀態，其文件系統仍然保存在磁盤上。
</details>

---

**Q14.** 在 Dockerfile 中，WORKDIR 指令用於聲明容器的工作目錄，後續的指令都會在此目錄下執行。_(對/錯)_

<details>
<summary>答案</summary>
<b>對</b>

<b>解析：</b>WORKDIR 指令設置容器的工作目錄，不僅影響後續 Dockerfile 指令的執行路徑，也會影響容器啟動後的默認工作目錄。如果目錄不存在，Docker 會自動創建。
</details>

---

**Q15.** 容器啟動後修改了容器裡的內容（如寫入數據到數據庫），容器退出後這些修改會自動保存到鏡像中。_(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b>

<b>解析：</b>容器不會自動保存修改到鏡像中。容器強調的是"用完就扔"的特性，容器內的修改在容器刪除後就會丟失。想要持久化數據，需要使用數據卷（volume）或掛載外部存儲。
</details>

---

## 第三部分：填充題 (每題 4 分，共 20 分)

**Q16.** Docker 使用 **\_\_\_\_** 文件系統作為存儲驅動，這是一種聯合文件系統技術。

<details>
<summary>答案</summary>
<b>overlay2</b>

<b>解析：</b>docker info 顯示存儲用的文件系統是 overlay2，這是一種聯合文件系統（Union FS）的實現技術，與 Linux 傳統的 ext3、ext4 文件系統不同，支持分層存儲，是 Docker 鏡像分層架構的基礎。
</details>

---

**Q17.** 在 Dockerfile 中，**\_\_\_\_** 指令用於執行 shell 命令，可以是安裝軟體、創建目錄、編譯程序等。

<details>
<summary>答案</summary>
<b>RUN</b>

<b>解析：</b>RUN 指令用於在構建鏡像時執行 shell 命令，例如安裝軟體包、創建目錄、編譯程序等。每個 RUN 指令都會創建一個新的鏡像層。
</details>

---

**Q18.** 使用 docker save 和 docker load 命令可以將鏡像導出為 **\_\_\_\_** 格式，方便保存和傳輸。

<details>
<summary>答案</summary>
<b>壓縮包 / tar / 歸檔文件</b>

<b>解析：</b>docker save 命令將鏡像導出為 tar 壓縮包格式，docker load 則從壓縮包導入鏡像。這樣可以在沒有網絡連接的環境中傳輸和部署鏡像。
</details>

---

**Q19.** 查看正在運行的容器列表使用 **\_\_\_\_** 命令，查看所有容器（包括已停止的）需要加上 **\_\_\_\_** 參數。

<details>
<summary>答案</summary>
<b>docker ps, -a</b>

<b>解析：</b>docker ps 顯示正在運行的容器列表，加上 -a（--all）參數則顯示所有容器，包括已停止的容器。
</details>

---

**Q20.** 使用 docker exec 命令進入運行中的容器時，需要添加 **\_\_\_\_** 參數來獲得交互式終端。

<details>
<summary>答案</summary>
<b>-it</b>

<b>解析：</b>docker exec -it [容器ID] sh 命令用於進入運行中的容器。-it 參數組合（interactive + tty）提供交互式終端環境，讓用戶可以在容器內執行命令。
</details>

---

## 第四部分：簡答題 (每題 10 分，共 20 分)

**Q21.** 請解釋容器與虛擬機的主要區別，以及容器為什麼啟動更快、資源利用率更高。

<details>
<summary>答案</summary>
<b>參考答案：</b>

容器與虛擬機的主要區別：

1. **架構層次不同：**
   - 容器是操作系統層級的虛擬化，容器本質上是操作系統裡的進程，加上了 namespace、cgroup、chroot 的限制
   - 虛擬機是硬件層級的虛擬化，需要完整的操作系統

2. **隔離程度不同：**
   - 容器共享宿主機內核，隔離的是進程、文件系統、網絡等資源
   - 虛擬機有獨立的內核和完整操作系統，隔離更徹底

3. **啟動更快、資源利用率更高的原因：**
   - 不需要啟動完整的操作系統和內核
   - 沒有硬件虛擬化的開銷
   - 共享宿主機內核，減少資源佔用
   - 容器與普通進程在資源使用方面沒有本質區別

<b>關鍵點：</b>
- 提到容器是進程，加上 namespace、cgroup、chroot 限制
- 說明共享宿主機內核
- 解釋無硬件虛擬化開銷
- 說明與普通進程資源使用無區別
</details>

---

**Q22.** 請描述 Dockerfile 中以下指令的作用：FROM、ENV、COPY、RUN、EXPOSE、WORKDIR，並說明它們的執行順序是否有影響。

<details>
<summary>答案</summary>
<b>參考答案：</b>

各指令的作用：

1. **FROM：** 指定構建的基礎鏡像，必須是 Dockerfile 的第一條指令
2. **ENV：** 定義環境變量，這些變量會在容器運行時以環境變量形式出現，供進程使用
3. **COPY：** 將構建上下文裡的文件拷貝到鏡像中，必須使用相對路徑
4. **RUN：** 在構建鏡像時執行 shell 命令，如安裝軟體、創建目錄等，每個 RUN 創建一個新層
5. **EXPOSE：** 聲明容器對外服務的端口號
6. **WORKDIR：** 設置容器的工作目錄，影響後續指令和容器啟動後的默認路徑

<b>執行順序的影響：</b>
- Dockerfile 中指令的執行順序非常重要
- FROM 必須是第一條指令
- WORKDIR 會影響後續 COPY、RUN 等指令的工作目錄
- ENV 定義的環境變量可以在後續指令中使用
- Docker 會按照指令順序依次執行，每條指令都會在上一條指令的基礎上創建新的鏡像層

<b>關鍵點：</b>
- 正確描述各指令作用
- 說明執行順序的重要性
- 提到每條指令會創建新層
- 說明指令之間的依賴關係
</details>

---

## 第五部分：配對題 (共 20 分)

**Q23.** 請將以下 Docker 命令與其功能進行配對：

| 命令             | 功能                                    |
| ---------------- | --------------------------------------- |
| 1. docker pull   | A. 從容器拷貝文件到宿主機               |
| 2. docker build  | B. 查看正在運行的容器                   |
| 3. docker cp     | C. 從鏡像倉庫下載鏡像                   |
| 4. docker ps     | D. 根據 Dockerfile 構建鏡像             |
| 5. docker exec   | E. 在運行中的容器內執行命令             |
| 6. docker save    | F. 啟動容器                             |
| 7. docker run     | G. 將鏡像導出為 tar 文件                |
| 8. docker images  | H. 查看本地鏡像列表                    |

<details>
<summary>答案</summary>
<b>配對結果：</b>

1-C (docker pull - 從鏡像倉庫下載鏡像)
2-D (docker build - 根據 Dockerfile 構建鏡像)
3-A (docker cp - 從容器拷貝文件到宿主機，或從宿主機拷貝文件到容器)
4-B (docker ps - 查看正在運行的容器)
5-E (docker exec - 在運行中的容器內執行命令)
6-G (docker save - 將鏡像導出為 tar 文件)
7-F (docker run - 啟動容器)
8-H (docker images - 查看本地鏡像列表)

<b>解析：</b>
- docker pull 從 Docker Hub 或其他鏡像倉庫下載鏡像到本地
- docker build 根據 Dockerfile 構建自定義鏡像
- docker cp 可雙向拷貝文件，支持宿主機到容器、容器到宿主機
- docker ps 顯示容器列表，-a 參數顯示所有容器
- docker exec 用於進入運行中的容器執行命令
- docker save 和 docker load 配合使用，用於鏡像的導出和導入
- docker run 從鏡像啟動容器
- docker images 列出本地所有鏡像
</details>

---

**Q24.** 請將以下 Docker 參數與其作用進行配對：

| 參數 | 作用                                     |
| ---- | ---------------------------------------- |
| 1. -d | A. 映射端口（宿主機端口:容器端口）       |
| 2. -v | B. 交互式終端，進入容器 shell            |
| 3. -p | C. 後台運行容器                           |
| 4. -it | D. 掛載卷，映射本地目錄到容器           |
| 5. --rm | E. 容器停止後自動刪除                   |
| 6. -t | F. 為鏡像打標籤                          |

<details>
<summary>答案</summary>
<b>配對結果：</b>

1-C (-d - 後台運行容器)
2-D (-v - 掛載卷，映射本地目錄到容器)
3-A (-p - 映射端口，格式為 宿主機端口:容器端口)
4-B (-it - 交互式終端，進入容器 shell)
5-E (--rm - 容器停止後自動刪除)
6-F (-t - 為鏡像打標籤，用於 docker build 命令)

<b>解析：</b>
- -d (detached) 讓容器在後台運行，不阻塞當前終端
- -v (volume) 用於數據持久化或共享文件，格式為 -v 宿主機路徑:容器路徑
- -p (publish) 映射端口，讓外部可以訪問容器服務
- -it 是 -i (interactive) 和 -t (tty) 的組合，提供交互式終端
- --rm 在測試或臨時容器中很有用，自動清理
- -t (tag) 在構建鏡像時指定名稱和標籤，格式為 -t name:tag
</details>

---

## 總分：100 分

---

## 評分標準

- **選擇題：** 每題 3 分，共 30 分
- **是非題：** 每題 2 分，共 10 分
- **填充題：** 每題 4 分，共 20 分
- **簡答題：** 每題 10 分，共 20 分
- **配對題：** 每組 10 分，共 20 分

---

## 建議學習重點

1. **Docker 基本命令：** 熟練掌握 docker version、docker info、docker ps、docker images、docker pull、docker run 等基本命令
2. **容器概念：** 理解容器是進程而非虛擬機，掌握 namespace、cgroup、chroot 的作用
3. **Dockerfile 指令：** 熟練使用 FROM、ENV、ARG、COPY、RUN、EXPOSE、WORKDIR 等指令
4. **容器與外部互通：** 掌握 docker cp、-v、-p 等與外部系統交互的方式
5. **鏡像管理：** 理解鏡像的構建、保存、加載過程

---

_生成自: Kubernetes 入門課程 - 08｜視頻：入門篇實操總結_