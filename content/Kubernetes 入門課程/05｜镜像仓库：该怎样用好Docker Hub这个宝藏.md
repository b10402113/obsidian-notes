# Docker Hub：鏡像倉庫的使用指南

## 課程概述
本課程介紹鏡像倉庫的概念與功能，詳細說明 Docker Hub 的使用方法，包括如何挑選合適鏡像、理解標籤命名規則，以及上傳自己鏡像的步骤。

## 核心觀念與實作解析

### 鏡像倉庫（Registry）是什麼？
- **Registry**：集中管理鏡像的服務站點，提供上傳、下載、查詢、删除等功能
- 形象比喻：如同「應用商店」或「档案馆」，分門別類存放容器化應用
- Docker默默使用官方的 **Docker Hub** 作為倉庫

### Docker Hub 概介
- 成立於 2014年，是世界上最大的鏡像倉庫
- 免費對公眾開放，任何人可上傳自己的鏡像
- 包含 Nginx、MongoDB、Node.js、Redis、OpenJDK 等下載量超10億的熱門應用

### Docker Hub 鏡像分類與挑選标准

| 分類 | 特點 | 可信度 |
|------|------|--------|
| **官方鏡像** | Docker 公司提供，經漏洞扫描與安全检测 | 最高，有「Official image」標記 |
| **認證鏡像** | 大公司（Bitnami、Rancher）發布，經Docker認證 | 高，有「Verified publisher」標記 |
| **半官方鏡像** | 公司官方發布但未認證（如OpenResty） | 中等 |
| **民間鏡像** | 個人上傳，測試不完全 | 低，需谨慎 |

挑選标准：
1. **優先選擇官方鏡像**
2. 參考**下載量**（百万级别為佳）
3. 查看**星数**與**更新历史**

### 鏡像命名規則
- **官方鏡像**：`應用名:標籤`（如 `nginx:alpine`）
- **非官方鏡像**：`用户名/應用名:標籤`（如 `bitnami/nginx:alpine`）

### 標籤命名含義解析
標籤格式通常為：`版本號 + 操作系統`

| 範例 | 含義 |
|------|------|
| `nginx:1.21.6-alpine` | 版本 1.21.6，基於 Alpine |
| `redis:7.0-rc-bullseye` | 版本 7.0候選版，基於 Debian 11（bullseye） |
| `node:17-buster-slim` | 版本 17，精簡版 Debian 10（buster） |

常用操作系統代號：
- Ubuntu 18.04 = `bionic`，Ubuntu 20.04 = `focal`
- Debian 9 = `stretch`，Debian 10 = `buster`，Debian 11 = `bullseye`

- `slim` = 精簡版（体积小，效率高）
- `fat` =完整版（工具多，適合開發调试）

### 上傳自己的鏡像到 Docker Hub
步骤：
1. 在 Docker Hub 注册用户
2. 本地登入：`docker login`
3.為鏡像打標籤：`docker tag ngx-app chronolaw/ngx-app:1.0`
4. 推送鏡像：`docker push chronolaw/ngx-app:1.0`

```bash
docker login                                    #登入 Docker Hub
docker tag ngx-app chronolaw/ngx-app:1.0        # 打標籤（加入用户名）
docker push chronolaw/ngx-app:1.0               # 推送鏡像
```

### 离线環境處理方案
1. **自建私有 Registry**：如 Docker Registry、CNCF Harbor
2. **使用 save/load 步驟**：導出鏡像為壓縮包

```bash
docker save ngx-app:latest -o ngx.tar           #導出鏡像
docker load -i ngx.tar                          #從壓縮包恢復鏡像
```

## 重點摘要
- Registry 是集中管理鏡像的服務站點，Docker Hub 是最大倉庫
- 鏡像分為官方、認證、半官方、民間四類，挑選時参考下載量與更新历史
- 非官方鏡像需加上用户名：`用户名/應用名:標籤`
- 標籤包含版本號與操作系統，`slim` 表示精簡版
- 上傳鏡像需先登入、打標籤、再推送；离线可用 save/load

## 關鍵字
Docker Hub, Registry, Official image, docker tag, docker push