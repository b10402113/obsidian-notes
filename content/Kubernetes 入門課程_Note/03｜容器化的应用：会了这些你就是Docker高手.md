# 容器化應用：Docker 鏡像與容器操作實戰

## 課程概述
本課程介紹「容器化應用」的概念，說明鏡像與容器的關係，並實際示範 Docker 鏡像與容器的常用操作命令，讓學員掌握 Docker 基本操作技能。

## 核心觀念與實作解析

### 什么是容器化的應用？
- **鏡像**：靜態的應用容器，將進程所需的文件系统、依賴庫、環境變數等打包固化
- **容器**：動態的應用鏡像，從鏡像啟動後運行的隔離環境

鏡像與容器**互相依存、互相轉化**：
- 鏡像 = 靜態打包文件（「樣板間」）
- 容器 = 鏡像的運行實例（從「樣板間」建造出的「實體房間」）

**容器化應用**：將應用封裝成鏡像，再交给容器環境運行，實現「一次編寫，到处運行」。

### 鏡像命名規則
鏡像完整名稱格式：`名字:標籤`

- **名字**：表明應用身份（如 `nginx`、`redis`、`alpine`）
- **標籤**：區分不同版本（如 `3.15`、`jammy`、`1.21-alpine`）
- **默認標籤**：未指定時使用 `latest`

```bash
docker pull alpine:3.15
docker pull ubuntu:jammy
docker pull nginx:1.21-alpine
docker pull redis                    # 默默使用 latest標籤
```

### 常用鏡像操作命令
| 命令 | 功能 |範例 |
|------|------|------|
| `docker pull` |拉取鏡像 | `docker pull nginx:alpine` |
| `docker images` | 列出本地鏡像 | `docker images` |
| `docker rmi` | 删除鏡像 | `docker rmi redis` 或 `docker rmi d4c`（用IMAGE ID前三位） |

**IMAGE ID**：鏡像唯一識別碼（十六進位），可使用前三位快速定位。

### 常用容器操作命令
`docker run` 是最複雜也最重要的命令，常用参数：

| 参数 | 功能 |
|------|------|
| `-it` |開啟交互式 Shell，進入容器内部 |
| `-d` |後台運行容器（適合 Nginx、Redis 等服務） |
| `--name` |為容器命名（方便查看） |
| `--rm` |容器結束後自動刪除 |

```bash
docker run -d nginx:alpine                        # 後台運行 Nginx
docker run -d --name red_srv redis               # 後台運行 Redis並命名
docker run -it --name ubuntu 2e6 sh              # 使用 IMAGE ID進入 Ubuntu
```

### 其他容器管理命令
| 命令 | 功能 |
|------|------|
| `docker exec` |在運行中的容器内執行程序 |
| `docker ps` | 列出運行中的容器 |
| `docker ps -a` |列出所有容器（含已停止） |
| `docker stop` |停止容器 |
| `docker rm` | 删除容器 |
| `docker start` | 重新啟動已停止的容器 |

```bash
docker exec -it red_srv sh           #進入 Redis 容器内部
docker stop ed4 d60 45c              # 停止多個容器（用CONTAINER ID）
docker rm ed d6 45                   # 删除多個容器
```

##重點摘要
- 鏡像打包應用及其運行環境，容器是鏡像的動態運行實例
- 鏡像命名格式為`名字:標籤`，默默使用 `latest`標籤
- 常用鏡像命令：`docker pull`、`docker images`、`docker rmi`
- 常用容器命令：`docker run`（最重要）、`docker exec`、`docker ps`、`docker stop`、`docker rm`

## 關鍵字
Image, Container, docker run, docker exec, IMAGE ID