# Dockerfile 與容器鏡像構建測驗

**科目：** Kubernetes 入門課程
**主題：** 創建容器鏡像：如何編寫正確、高效的 Dockerfile
**難度：** 混合（初級 50%、中級 40%、高級 10%）
**題目數：** 19 題
**建議時間：** 25 分鐘

---

## 第一部分：選擇題（每題 5 分，共 25 分）

**Q1.** 容器鏡像內部的分層結構（Layer）有什麼重要作用？

A) 提高容器啟動速度
B) 減少存儲空間和網絡傳輸成本，相同層可在不同鏡像間共享
C) 增強容器安全性
D) 簡化 Dockerfile 編寫

<details>
<summary>答案</summary>
<b>B)</b> 鏡像分層的主要優勢是減少存儲空間和網絡傳輸成本。相同的 Layer 可以在不同鏡像之間共享，避免重複存儲相同的文件系統內容。例如，多個基於 Ubuntu 的鏡像可以共享同一份 Ubuntu 根目錄層，而不是各自存儲一份副本。
</details>

**Q2.** 關於 Dockerfile 中的 FROM 指令，以下哪項描述是正確的？

A) FROM 指令可以省略，Docker 會自動選擇基礎鏡像
B) FROM 指令必須是 Dockerfile 的第一條指令，用於選擇基礎鏡像
C) FROM 指令可以放在 Dockerfile 的任意位置
D) FROM 指令用於聲明容器服務端口

<details>
<summary>答案</summary>
<b>B)</b> FROM 指令必須是 Dockerfile 的第一條指令，用於選擇構建使用的基礎鏡像，相當於"打地基"。所有 Dockerfile 都必須從 FROM 開始，它指定了後續構建步驟的基礎環境。
</details>

**Q3.** 在 Dockerfile 中，以下哪個指令會生成新的鏡像層（Layer）？

A) ENV
B) ARG
C) RUN
D) EXPOSE

<details>
<summary>答案</summary>
<b>C)</b> 只有 RUN、COPY 和 ADD 指令會生成新的鏡像層。其他指令（如 ENV、ARG、EXPOSE）只會產生臨時的中間層，不會增加構建大小。這也是為什麼在編寫 Dockerfile 時應該精簡合併 RUN 指令，避免產生過多層。
</details>

**Q4.** 關於 docker build 命令的"構建上下文"（build context），以下哪項描述是正確的？

A) 構建上下文就是 Dockerfile 所在的目錄
B) 構建上下文與 Dockerfile 必須在同一目錄
C) 構建上下文會被打包上傳到 Docker daemon，決定了 COPY 指令可訪問的文件範圍
D) 構建上下文只能是當前目錄

<details>
<summary>答案</summary>
<b>C)</b> 構建上下文會被打包上傳到 Docker daemon，它決定了 COPY 指令可以訪問的文件範圍。因為 Docker 採用客戶端-服務器架構，docker 客戶端需要將相關文件打包發送給服務器端的 Docker daemon。COPY 命令只能使用基於構建上下文的相對路徑，不能訪問上下文之外的文件。
</details>

**Q5.** 在 Dockerfile 中，ARG 和 ENV 指令的主要區別是什麼？

A) ARG 用於定義數值，ENV 用於定義字符串
B) ARG 創建的變量只在構建過程可見，ENV 創建的變量在構建和運行時都可見
C) ARG 和 ENV 完全相同，可以互換使用
D) ENV 只能用於 FROM 指令

<details>
<summary>答案</summary>
<b>B)</b> ARG 創建的變量只在鏡像構建過程中可見，容器運行時不可見；而 ENV 創建的變量不僅能在構建鏡像的過程中使用，在容器運行時也能以環境變量的形式被應用程序使用。ARG 通常用於構建時參數（如基礎鏡像版本），ENV 用於運行時配置。
</details>

---

## 第二部分：是非題（每題 4 分，共 20 分）

**Q6.** 容器鏡像中的每個 Layer 都是可讀寫的，可以隨時修改。 _(是/否)_

<details>
<summary>答案</summary>
<b>否</b> - 容器鏡像中的每個 Layer 都是<b>只讀不可修改</b>的文件層。當容器運行時，Docker 會在鏡像的最上層添加一個可讀寫層，容器的寫操作通過 Copy-on-Write（寫時複製）機制記錄在這個可寫層中，下層的只讀鏡像層保持不變。
</details>

**Q7.** Dockerfile 中的指令越多越好，可以更詳細地描述構建過程。 _(是/否)_

<details>
<summary>答案</summary>
<b>否</b> - 應該盡量精簡合併 Dockerfile 指令。因為每個指令都會生成一個鏡像層，過多的層會導致鏡像臃腫不堪，影響存儲、傳輸和加載效率。應該使用 \ 和 && 將相關的 Shell 命令合併到一個 RUN 指令中執行。
</details>

**Q8.** COPY 指令可以從本機的任意路徑拷貝文件到鏡像中。 _(是/否)_

<details>
<summary>答案</summary>
<b>否</b> - COPY 指令只能從"構建上下文"路徑中拷貝文件，不能隨意指定本機任意路徑。構建上下文之外的文件對 Docker daemon 是不可見的，因為 docker build 時只會將構建上下文目錄打包上傳給服務器。
</details>

**Q9.** EXPOSE 指令會自動在宿主機上開放相應的端口。 _(是/否)_

<details>
<summary>答案</summary>
<b>否</b> - EXPOSE 指令只是<b>聲明</b>容器對外服務的端口號，作為文檔和說明用途，不會自動在宿主機上映射端口。實際的端口映射需要在 docker run 時使用 -p 參數明確指定。EXPOSE 主要用於告知開發者容器會使用哪些端口。
</details>

**Q10.** 使用 .dockerignore 文件可以排除不需要的文件，提高構建效率。 _(是/否)_

<details>
<summary>答案</summary>
<b>是</b> - .dockerignore 文件的語法與 .gitignore 類似，可以排除不需要打包上傳到 Docker daemon 的文件（如 .git、.svn、readme 等）。這樣可以減少構建上下文的大小，提高網絡傳輸效率和構建速度。
</details>

---

## 第三部分：填充題（每題 5 分，共 25 分）

**Q11.** 容器鏡像採用 **\_\_\_\_** 技術將多個只讀層合併成容器最終看到的文件系統。

<details>
<summary>答案</summary>
<b>Union FS（聯合文件系統）</b>

Union FS 是容器鏡像的核心技術，它將多個只讀的鏡像層堆疊起來，像搭積木一樣組合成完整的文件系統。當不同層在同一位置都有文件時，上層的文件會覆蓋下層的同名文件。
</details>

**Q12.** Dockerfile 中使用 **\_\_\_\_** 指令來執行 Shell 命令，它是 Dockerfile 中最複雜、最靈活的指令。

<details>
<summary>答案</summary>
<b>RUN</b>

RUN 指令可以執行任意的 Shell 命令，例如更新系統、安裝應用、下載文件、創建目錄、編譯程序等。為了減少層數，通常使用 \ 和 && 將多個命令合併到一個 RUN 指令中。
</details>

**Q13.** Docker 採用 **\_\_\_\_** 架構，docker 命令行是客戶端，實際的鏡像構建工作由 Docker daemon 完成。

<details>
<summary>答案</summary>
<b>客戶端-服務器（Client-Server）</b>

因為採用客戶端-服務器架構，docker 客戶端需要將構建上下文打包上傳給 Docker daemon，服務器端才能訪問這些文件。這就是為什麼 COPY 只能使用構建上下文內的文件路徑。
</details>

**Q14.** 使用 docker build 命令時，**\_\_\_\_** 參數用於指定 Dockerfile 文件名，**\_\_\_\_** 參數用於為構建的鏡像命名。

<details>
<summary>答案</summary>
<b>-f，-t</b>

-f 參數用於指定 Dockerfile 文件名（如果省略則使用當前目錄下名為 Dockerfile 的文件）。-t 參數用於指定鏡像的標籤（tag），格式為 name:tag，如果不提供標籤則默認為 latest。
</details>

**Q15.** 為了避免鏡像層數過多，在 Dockerfile 中應該使用 **\_\_\_\_** 符號進行續行，使用 **\_\_\_\_** 符號連接多個命令。

<details>
<summary>答案</summary>
<b>\（反斜線），&&</b>

在 Dockerfile 中，使用 \ 符號可以將一條指令分成多行書寫，使用 && 符號可以連接多個 Shell 命令，這樣可以在邏輯上保持為一個指令，從而只生成一個鏡像層。
</details>

---

## 第四部分：簡答題（每題 10 分，共 20 分）

**Q16.** 解釋鏡像分層（Layer）機制如何實現層共享，並說明這帶來了哪些好處。

<details>
<summary>答案</summary>
<b>層共享機制：</b>

容器鏡像內部由多個只讀的 Layer 組成，每個 Layer 都有唯一的標識。當多個鏡像使用相同的基礎層時，Docker 不會重複存儲這些層，而是在本地只保留一份副本，讓多個鏡像共享這些相同的層。

<b>帶來的好處：</b>

1. <b>節省存儲空間：</b> 相同的層只需存儲一次，避免了大量重複數據。例如，一千個基於 Ubuntu 的鏡像只需要一份 Ubuntu 根目錄層。

2. <b>減少網絡傳輸：</b> docker pull 時會檢查本地是否已存在某些層，已存在的層不會重複下載，節省帶寬和時間。

3. <b>加速構建過程：</b> 當修改 Dockerfile 時，只有受影響的層需要重新構建，未改變的層可以使用緩存。

4. <b>快速分發：</b> 由於層的共享特性，鏡像的分發更加高效，適合大規模容器化部署。

5. <b>增量更新：</b> 應用更新時只需拉取新的層，不需要重新下載整個鏡像。
</details>

**Q17.** 簡述 docker build 的執行流程，並說明為什麼需要"構建上下文"。

<details>
<summary>答案</summary>
<b>docker build 執行流程：</b>

1. <b>解析參數：</b> 讀取 -f 參數指定的 Dockerfile 文件（或默認的 Dockerfile）。

2. <b>打包上下文：</b> 將構建上下文目錄中的所有文件（排除 .dockerignore 中指定的文件）打包。

3. <b>上傳到 daemon：</b> docker 客戶端將打包的構建上下文上傳給 Docker daemon（顯示 "Sending build context to Docker daemon" 信息）。

4. <b>逐行執行：</b> Docker daemon 逐行讀取並執行 Dockerfile 中的指令，每條指令生成一個臨時容器，執行完成後提交為一個新的鏡像層。

5. <b>生成鏡像：</b> 所有指令執行完成後，生成最終的鏡像，如果指定了 -t 參數則為鏡像打上標籤。

<b>為什麼需要構建上下文：</b>

1. <b>架構隔離：</b> Docker 採用客戶端-服務器架構，docker 命令行是客戶端，實際構建工作由 Docker daemon（服務器）完成。

2. <b>文件訪問限制：</b> Docker daemon 運行在獨立的環境中，無法直接訪問客戶端本地的文件系統，只能訪問客戶端上傳的文件。

3. <b>確定性構建：</b> 通過明確定義構建上下文，保證了構建過程的可重現性，避免依賴本地環境中的意外文件。

4. <b>安全性：</b> 限制了 Docker daemon 可訪問的文件範圍，防止意外或惡意地將敏感文件打包進鏡像。

COPY 指令只能使用構建上下文內的相對路徑，正是因為 Docker daemon 只能看到上傳的構建上下文中的文件。
</details>

---

## 第五部分：配對題（10 分）

**Q18.** 將以下 Dockerfile 指令與其功能進行配對：

| 指令        | 功能描述                          |
| ----------- | --------------------------------- |
| 1. FROM     | A. 聲明容器對外服務的端口號        |
| 2. COPY     | B. 定義只在構建過程中可見的變量    |
| 3. RUN      | C. 選擇構建使用的基礎鏡像          |
| 4. ARG      | D. 從構建上下文拷貝文件到鏡像       |
| 5. EXPOSE   | E. 執行 Shell 命令，生成鏡像層      |

<details>
<summary>答案</summary>
1-C, 2-D, 3-E, 4-B, 5-A

<b>解析：</b>

- <b>FROM：</b> 必須是 Dockerfile 的第一條指令，選擇構建使用的基礎鏡像，相當於"打地基"。
- <b>COPY：</b> 從構建上下文拷貝文件到鏡像的指定路徑，語法類似 Linux 的 cp 命令。
- <b>RUN：</b> 執行任意的 Shell 命令，是 Dockerfile 中最靈活的指令，會生成新的鏡像層。
- <b>ARG：</b> 定義構建時變量，只在鏡像構建過程中可見，容器運行時不可見。
- <b>EXPOSE：</b> 聲明容器對外服務的端口號，主要作為文檔說明用途。
</details>

---

## 第六部分：綜合應用題（10 分）

**Q19.** 閱讀以下 Dockerfile，解釋每個指令的含義，並指出可以優化的地方：

```dockerfile
ARG IMAGE_BASE="nginx"
ARG IMAGE_TAG="1.21-alpine"

FROM ${IMAGE_BASE}:${IMAGE_TAG}

COPY ./default.conf /etc/nginx/conf.d/

RUN cd /usr/share/nginx/html
RUN echo "hello nginx" > a.txt

EXPOSE 8081 8082 8083

ENV PATH=$PATH:/tmp
ENV DEBUG=OFF
```

<details>
<summary>答案</summary>
<b>指令含義解析：</b>

1. <b>ARG IMAGE_BASE="nginx"</b> - 定義構建時變量 IMAGE_BASE，默認值為 "nginx"，可用於 FROM 指令。

2. <b>ARG IMAGE_TAG="1.21-alpine"</b> - 定義構建時變量 IMAGE_TAG，默認值為 "1.21-alpine"。

3. <b>FROM ${IMAGE_BASE}:${IMAGE_TAG}</b> - 使用變量指定基礎鏡像為 nginx:1.21-alpine。

4. <b>COPY ./default.conf /etc/nginx/conf.d/</b> - 將構建上下文中的 default.conf 拷貝到鏡像的 /etc/nginx/conf.d/ 目錄。

5. <b>RUN cd /usr/share/nginx/html</b> - 切換到 /usr/share/nginx/html 目錄（但這個命令單獨執行沒有意義）。

6. <b>RUN echo "hello nginx" > a.txt</b> - 在當前目錄創建 a.txt 文件並寫入內容。

7. <b>EXPOSE 8081 8082 8083</b> - 聲明容器會使用 8081、8082、8083 三個端口。

8. <b>ENV PATH=$PATH:/tmp</b> - 設置環境變量 PATH，將 /tmp 添加到現有 PATH 中。

9. <b>ENV DEBUG=OFF</b> - 設置環境變量 DEBUG 為 OFF。

<b>可優化的地方：</b>

1. <b>合併 RUN 指令：</b> 第 5、6 行的 RUN 指令應該合併為一個，避免生成多個不必要的鏡像層：
   ```dockerfile
   RUN cd /usr/share/nginx/html && echo "hello nginx" > a.txt
   ```

2. <b>合併 ENV 指令：</b> 兩個 ENV 指令可以合併為一個，雖然 ENV 不會增加層大小，但可以簡化 Dockerfile：
   ```dockerfile
   ENV PATH=$PATH:/tmp \
       DEBUG=OFF
   ```

3. <b>移除無效命令：</b> 單獨的 `RUN cd` 命令在下一個 RUN 指令時不會保持當前目錄，因為每個 RUN 都會在新的 shell 中執行。應該使用 WORKDIR 指令或將 cd 和後續命令合併。

<b>優化後的版本：</b>

```dockerfile
ARG IMAGE_BASE="nginx"
ARG IMAGE_TAG="1.21-alpine"

FROM ${IMAGE_BASE}:${IMAGE_TAG}

COPY ./default.conf /etc/nginx/conf.d/

RUN cd /usr/share/nginx/html && echo "hello nginx" > a.txt

EXPOSE 8081 8082 8083

ENV PATH=$PATH:/tmp \
    DEBUG=OFF
```
</details>

---

## 總結

本測驗涵蓋了以下核心概念：

- ✅ 容器鏡像的 Layer 分層機制與 Union FS
- ✅ Dockerfile 基本指令（FROM、COPY、RUN、EXPOSE、ARG、ENV）
- ✅ Docker build 工作流程與構建上下文
- ✅ 鏡像優化原則（減少層數、合併指令）
- ✅ .dockerignore 文件的作用
- ✅ ARG 與 ENV 的區別
- ✅ 實際 Dockerfile 編寫與優化技巧

---

_本測驗生成自：04｜创建容器镜像：如何编写正确、高效的Dockerfile_