# Docker 容器化應用測驗

**科目：** Kubernetes 入門實戰課程
**主題：** 容器化的應用：會了這些你就是 Docker 高手
**難度：** 混合（簡單、中等、困難）
**題數：** 20 題
**建議時間：** 30 分鐘

---

## 第一部分：選擇題（每題 5 分，共 30 分）

**Q1.** 關於容器鏡像（Image）的描述，下列哪一項是**錯誤**的？

A) 鏡像是只讀的，不允許修改
B) 鏡像打包了應用程序及其完整的運行環境
C) 鏡像與容器是互相依存、互相轉化的關係
D) 鏡像只能在創建它的操作系統上運行，不具有跨平台性

<details>
<summary>答案</summary>
<b>D)</b> 鏡像具有非常好的跨平台便攜性和兼容性，可以讓開發者在一個系統上開發（例如 Ubuntu），打包成鏡像後，在另一個系統上運行（例如 CentOS），完全不需要考慮環境依賴的問題。這正是容器技術的優勢所在。
</details>

---

**Q2.** 執行 `docker pull nginx` 命令時，如果沒有明確指定標籤（tag），Docker 會使用哪個默認標籤？

A) stable
B) latest
C) current
D) default

<details>
<summary>答案</summary>
<b>B)</b> 如果只提供鏡像名字而沒有附帶標籤，Docker 會使用默認的 "latest" 標籤。鏡像的完整名字由兩個部分組成：名字和標籤，中間用 : 連接。
</details>

---

**Q3.** 關於 IMAGE ID 和 CONTAINER ID 的描述，下列哪一項是**正確**的？

A) IMAGE ID 和 CONTAINER ID 都是隨機生成的，沒有實際用途
B) IMAGE ID 和 CONTAINER ID 都是唯一的標識，可以使用前幾位數字進行"短路"操作
C) 同一個鏡像的不同標籤必定會有不同的 IMAGE ID
D) CONTAINER ID 每次重啟容器後都會改變

<details>
<summary>答案</summary>
<b>B)</b> IMAGE ID 和 CONTAINER ID 都是唯一的標識，就像身份證號一樣。Docker 提供了"短路"操作，在本地使用時通常只需要寫出前三位就能夠快速定位。同一個鏡像可以打上不同的標籤，但它們的 IMAGE ID 是相同的。
</details>

---

**Q4.** 下列哪個命令組合可以讓你進入一個正在運行的 Redis 容器並打開 Shell？

A) `docker run -it redis sh`
B) `docker exec -d redis sh`
C) `docker exec -it redis sh`
D) `docker attach -it redis sh`

<details>
<summary>答案</summary>
<b>C)</b> 對於正在運行中的容器，我們使用 <b>docker exec</b> 命令在裡面執行另一個程序。使用 <b>-it</b> 參數可以開啟一個交互式操作的 Shell，從而進入容器內部。選項 A 的 docker run 會創建新容器，而不是進入已存在的容器。
</details>

---

**Q5.** 執行 `docker run -d --name web_srv nginx:alpine` 命令後，下列哪項描述是**正確**的？

A) 容器會在前台運行，並自動進入容器的 Shell
B) 容器會在後台運行，可以被命名為 web_srv
C) 容器運行完畢後會自動刪除
D) 需要手動啟動容器才能運行

<details>
<summary>答案</summary>
<b>B)</b> <b>-d</b> 參數表示讓容器在後台運行，這在啟動 Nginx、Redis 等服務器程序時非常有用。<b>--name</b> 參數為容器命名為 web_srv，方便後續查看和管理。如果沒有 <b>--rm</b> 參數，容器運行完畢後不會自動刪除。
</details>

---

**Q6.** 關於 `docker rmi` 和 `docker rm` 命令的區別，下列描述哪項是**正確**的？

A) 兩個命令功能完全相同，可以互換使用
B) docker rmi 刪除容器，docker rm 刪除鏡像
C) docker rmi 刪除鏡像，docker rm 刪除容器
D) docker rmi 只能用鏡像名，docker rm 只能用 ID

<details>
<summary>答案</summary>
<b>C)</b> <b>docker rmi</b> 是 "remove image" 的簡寫，用來刪除不再使用的鏡像。<b>docker rm</b> 則用來刪除容器，注意它沒有後面的字母 "i"，所以只會刪除容器，不刪除鏡像。
</details>

---

## 第二部分：是非題（每題 4 分，共 20 分）

**Q7.** 鏡像是容器的靜態形式，容器是鏡像的動態形式。_(對/錯)_

<details>
<summary>答案</summary>
<b>對</b> - 鏡像是靜態的應用容器，容器是動態的應用鏡像，兩者互相依存、互相轉化、密不可分。鏡像把運行進程所需的所有信息打包整合，操作系統能根據鏡像快速重建容器。
</details>

---

**Q8.** 使用 `docker stop` 命令停止容器後，容器會被徹底銷毀，無法再次啟動。_(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - 容器被停止後使用 <b>docker ps</b> 命令看不到，但容器並沒有被徹底銷毀。可以使用 <b>docker ps -a</b> 命令查看所有容器（包括已停止的），並使用 <b>docker start</b> 命令再次啟動。只有使用 <b>docker rm</b> 才會徹底刪除容器。
</details>

---

**Q9.** `docker run -it alpine sh` 和 `docker exec -it alpine sh` 這兩個命令的效果完全相同。_(對/錯)_

<details>
<summary>答案</summary>
<b>錯</b> - <b>docker run</b> 是創建並啟動一個新容器，而 <b>docker exec</b> 是在已經存在的容器裡執行程序。docker run 會創建新的容器，docker exec 不會創建新容器。兩者的使用場景完全不同。
</details>

---

**Q10.** 同一個鏡像可以有多個不同的標籤，但它們的 IMAGE ID 是相同的。_(對/錯)_

<details>
<summary>答案</summary>
<b>對</b> - 這就像人的身份證號碼是唯一的，但可以有大名、小名、暱稱、綽號。同一個鏡像可以打上不同的標籤，這樣應用在不同場合就更容易理解，但它們的 IMAGE ID 是相同的。
</details>

---

**Q11.** 使用 `--rm` 參數運行容器時，容器會在停止後自動刪除，無需手動執行 `docker rm` 命令。_(對/錯)_

<details>
<summary>答案</summary>
<b>對</b> - 在執行 <b>docker run</b> 命令時加上 <b>--rm</b> 參數，會告訴 Docker 不保存容器，只要運行完畢就自動清除，省去了手工管理容器的麻煩。
</details>

---

## 第三部分：填充題（每題 6 分，共 24 分）

**Q12.** 鏡像的完整名字由 **____** 和 **____** 兩個部分組成，中間用 **____** 符號連接。

<details>
<summary>答案</summary>
<b>名字、標籤、:</b> 鏡像的完整名字格式為 name:tag，名字表明了應用的身份（如 busybox、nginx），標籤則是為了區分不同版本而做的額外標記。
</details>

---

**Q13.** Docker 命令中，**____** 參數表示開啟交互式操作的 Shell，**____** 參數表示讓容器在後台運行，**____** 參數可以為容器指定名稱。

<details>
<summary>答案</summary>
<b>-it、-d、--name</b> - <b>-it</b> 是 -i 和 -t 兩個參數的組合，用於開啟交互式操作；<b>-d</b> 讓容器在後台運行，適用於服務器程序；<b>--name</b> 為容器命名，方便管理。
</details>

---

**Q14.** 查看當地所有鏡像的命令是 **____**，查看正在運行的容器的命令是 **____**，查看所有容器（包括已停止）的命令是 **____**。

<details>
<summary>答案</summary>
<b>docker images、docker ps、docker ps -a</b> - <b>docker images</b> 列出本地已有的鏡像；<b>docker ps</b> 查看正在運行的容器；<b>docker ps -a</b> 查看所有容器，包括已停止的。
</details>

---

**Q15.** 容器化的應用是指應用程序不再直接和操作系統打交道，而是封裝成 **____**，再交給 **____** 去運行，實現了"一次編寫，到處運行"的理念。

<details>
<summary>答案</summary>
<b>鏡像、容器環境</b> - 容器化的應用就是指應用程序以鏡像的形式打包，然後在容器環境裡從鏡像啟動容器。鏡像打包了應用程序的所有運行依賴項，讓應用可以在任何環境中一致運行。
</details>

---

## 第四部分：簡答題（每題 10 分，共 20 分）

**Q16.** 請說明容器鏡像與 rpm、deb 等傳統安裝包的主要區別及其優缺點。

<details>
<summary>答案</summary>
<b>主要區別：</b>

鏡像與 rpm、deb 等安裝包都打包了應用程序，但最大的不同在於鏡像裡不僅有基本的可執行文件，還有應用運行時的整個系統環境。

<b>鏡像的優點：</b>
- 具有非常好的跨平台便攜性和兼容性
- 開發者在一個系統上開發，打包成鏡像後可在另一個系統上運行
- 完全不需要考慮環境依賴的問題
- 是一種更高級的應用打包方式
- 實現了"一次編寫，到處運行"的理念

<b>傳統安裝包的特點：</b>
- 不同 Linux 發行版需要不同的安裝包格式
- 依賴系統環境，可能遇到依賴衝突問題
- 文件相對較小，但需要額外安裝依賴庫

<b>鏡像的缺點：</b>
- 鏡像文件通常比傳統安裝包大，因為包含了完整的運行環境
- 對於簡單應用可能過於"重量級"
</details>

---

**Q17.** 請詳細說明 `docker run` 和 `docker exec` 命令的區別，以及它們各自的適用場景。

<details>
<summary>答案</summary>
<b>主要區別：</b>

<b>docker run：</b>
- 用於從鏡像創建並啟動一個新的容器
- 每次執行都會創建一個新的容器實例
- 有豐富的啟動參數，如掛載 volume、端口映射等
- 是容器運行啟動的基礎命令

<b>docker exec：</b>
- 在已經運行的容器內執行程序
- 不會創建新的容器，而是在現有容器中執行命令
- 容器重啟後，之前 exec 的 session 將失效
- 主要用於調試、排錯和管理運行中的容器

<b>適用場景：</b>

<b>docker run 適用於：</b>
- 首次啟動一個應用服務
- 創建新的容器實例進行測試
- 使用鏡像啟動應用程序

<b>docker exec 適用於：</b>
- 進入運行中的容器查看狀態
- 在容器內執行調試命令
- 查看容器內的服務運行狀態或日誌
- 對運行中的容器進行故障排查

<b>示例對比：</b>
- `docker run -d nginx` → 創建並啟動新的 Nginx 容器
- `docker exec -it nginx sh` → 進入已運行的 Nginx 容器
</details>

---

## 第五部分：配對題（每個配對 1 分，共 6 分）

**Q18.** 請將下列 Docker 命令與其功能進行配對：

| 命令 | 功能 |
|------|------|
| 1. docker pull | A. 刪除容器 |
| 2. docker rmi | B. 拉取鏡像 |
| 3. docker rm | C. 在容器內執行命令 |
| 4. docker exec | D. 列出本地鏡像 |
| 5. docker images | E. 刪除鏡像 |
| 6. docker stop | F. 停止運行中的容器 |

<details>
<summary>答案</summary>
1-B, 2-E, 3-A, 4-C, 5-D, 6-F

<b>解析：</b>
- <b>docker pull：</b> 從遠端倉庫拉取鏡像到本地
- <b>docker rmi：</b> 刪除不再使用的鏡像，節約磁盤空間（rmi = remove image）
- <b>docker rm：</b> 徹底刪除容器（注意：沒有字母 "i"）
- <b>docker exec：</b> 在已運行的容器內執行程序，常用於調試
- <b>docker images：</b> 列出當前本地已有的鏡像
- <b>docker stop：</b> 強制停止運行中的容器
</details>

---

## 答案總結

**選擇題：** 1-D, 2-B, 3-B, 4-C, 5-B, 6-C

**是非題：** 7-對, 8-錯, 9-錯, 10-對, 11-對

**填充題：**
- Q12: 名字、標籤、:
- Q13: -it、-d、--name
- Q14: docker images、docker ps、docker ps -a
- Q15: 鏡像、容器環境

**簡答題：** 見各題詳細答案

**配對題：** 1-B, 2-E, 3-A, 4-C, 5-D, 6-F

---

**生成來源：** Kubernetes 入門實戰課程 - 03｜容器化的應用：會了這些你就是 Docker 高手

**學習重點回顧：**
1. 鏡像是容器的靜態形式，容器是鏡像的動態形式
2. 常用鏡像操作：docker pull、docker images、docker rmi
3. 常用容器操作：docker run、docker exec、docker ps、docker stop、docker rm
4. 重要參數：-it（交互式）、-d（後台）、--name（命名）、--rm（自動刪除）
5. docker run 創建新容器，docker exec 在現有容器中執行命令