# Dockerfile：正確高效的鏡像构建指南

## 課程概述
本課程深入剖析容器鏡像的内部分層結構，並詳細說明 Dockerfile 的编写方法與最佳實踐，讓學員掌握如何构建自己的容器鏡像。

## 核心觀念與實作解析

### 鏡像的内部機制：分層結構（Layer）
- **鏡像並非平坦結構**，而是由多個「Layer」堆疊組成
- 每個 Layer 都是**只读不可修改**的一組文件
- 相同的 Layer 可在不同鏡像間**共享**，節省儲存與傳輸成本
- 使用 **Union FS 聯合文件系統**將多層合并成容器看到的文件系统

比喻：如同「千层糕」，每層糕就是一個 Layer，上层文件會屏蔽下层同位置文件。

查看鏡像分層資訊：
```bash
docker inspect nginx:alpine        # RootFS部分顯示Layer數量
```

### Dockerfile 是什麼？
- **Dockerfile 是「施工圖紙」**，記錄一系列构建指令
- 每個指令會生成一個 Layer
- Docker順序執行所有步驟，最终創建出新鏡像

最簡範例：
```dockerfile
FROM busybox                #選擇基础鏡像（打地基）
CMD echo "hello world"      #啟動容器時默默運行的命令
```

### docker build 构建鏡像
```bash
docker build -f Dockerfile.busybox .        # -f指定Dockerfile，.為构建上下文
docker build -t ngx-app:1.0 .               # -t為鏡像打標籤
```

**构建上下文（build context）**：要打包進鏡像的文件所在目錄，會打包上傳給 Docker daemon。

### Dockerfile 常用指令

| 指令 | 功能 | 範例 |
|------|------|------|
| `FROM` |選擇基础鏡像（必須為第一條） | `FROM alpine:3.15` |
| `COPY` | 拷貝文件到鏡像（需在构建上下文内） | `COPY ./a.txt /tmp/a.txt` |
| `RUN` | 執行 Shell 命令（最靈活） | `RUN apt-get update && apt-get install -y curl` |
| `ARG` | 定义构建時變數（容器運行時不可見） | `ARG IMAGE_BASE="node"` |
| `ENV` | 定义環境變數（構建與運行時都可見） | `ENV DEBUG=OFF` |
| `EXPOSE` | 声明對外服務端口 | `EXPOSE 443` |
| `CMD` | 容器啟動默默命令 | `CMD ["nginx", "-g", "daemon off;"]` |

### Dockerfile 最佳實踐
1. **基礎鏡像選擇**：
   - Alpine：鏡像小、安全性高
   - Ubuntu/Debian/CentOS：穩定性佳

2. **RUN 指令优化**：
   - 使用 `\` 續行符與 `&&` 連接，保持逻辑一行
   - 或將命令集中到脚本文件，用COPY拷貝後RUN執行

```dockerfile
RUN apt-get update \
    && apt-get install -y build-essential curl \
    && cd /tmp && curl -fSL xxx.tar.gz -o xxx.tar.gz \
    && tar xzf xxx.tar.gz && make && make clean
```

3. **减少 Layer 數量**：避免滥用指令，盡量精簡合并

4. **使用 .dockerignore**：排除不需要的文件（如 `.git`、`.svn`）

```dockerignore
*.swp
*.sh
```

## 重點摘要
- 鏡像由多個只读 Layer 組成，相同 Layer 可共享，節省資源
- Dockerfile 是构建鏡像的「施工圖紙」，每條指令生成一個 Layer
- `FROM` 必須為第一條指令，選擇基础鏡像
- 常用指令：`COPY`、`RUN`、`ARG`、`ENV`、`EXPOSE`
- 构建時應盡量减少 Layer 數量，使用 `-t` 為鏡像命名

## 關鍵字
Dockerfile, Layer, Union FS, docker build, build context