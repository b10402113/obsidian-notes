# Docker實戰演练：Registry與 WordPress部署

## 課程概述
本課程為「入門篇」總結，實際演示使用 Docker 搭建私有鏡像倉庫 Registry與WordPress 网站，並指出容器技术的局限，引出容器编排與 Kubernetes 的需求。

## 核心觀念與實作解析

### 容器技术核心概念回顧
容器技术三大核心：
- **Container**：被隔離的進程，運行在沙盒環境
- **Image**：静态的應用容器，打包應用與完整運行環境
- **Registry**：鏡像倉庫，集中管理鏡像的分發與存取

### 实戰一：搭建私有鏡像倉庫 Registry

#### 拉取與啟動 Registry
```bash
docker pull registry                            #拉取官方 Registry鏡像
docker run -d -p 5000:5000 registry             #啟動，映射5000端口
docker ps                                       # 查看運行狀態
```

#### 上傳鏡像到私有倉庫
```bash
docker tag nginx:alpine 127.0.0.1:5000/nginx:alpine    # 打標籤（加入倉庫地址）
docker push 127.0.0.1:5000/nginx:alpine                # 推送到私有倉庫
docker rmi 127.0.0.1:5000/nginx:alpine                 # 删除本地標籤
docker pull 127.0.0.1:5000/nginx:alpine                # 從私有倉庫拉取验证
```

#### 使用 API 查看倉庫内容
```bash
curl 127.1:5000/v2/_catalog                    # 查看鏡像列表
curl 127.1:5000/v2/nginx/tags/list             # 查看特定鏡像的標籤列表
```

### 实戰二：搭建 WordPress 网站

系統架構：**Nginx → WordPress → MariaDB**

#### 1. 啟動 MariaDB 数据庫
```bash
docker run -d --rm \
    --env MARIADB_DATABASE=db \
    --env MARIADB_USER=wp \
    --env MARIADB_PASSWORD=123 \
    --env MARIADB_ROOT_PASSWORD=123 \
    mariadb:10
```

验证数据庫：
```bash
docker exec -it 9ac mysql -u wp -p              #進入 MySQL
show databases;                                 # 查看数据库
```

MariaDB 容器 IP：`172.17.0.2`

#### 2. 啟動 WordPress應用
```bash
docker run -d --rm \
    --env WORDPRESS_DB_HOST=172.17.0.2 \        # MariaDB IP地址
    --env WORDPRESS_DB_USER=wp \
    --env WORDPRESS_DB_PASSWORD=123 \
    --env WORDPRESS_DB_NAME=db \
    wordpress:5
```

WordPress 容器 IP：`172.17.0.3`

#### 3. 配置 Nginx 反向代理
Nginx配置文件 `wp.conf`：
```nginx
server {
    listen 80;
    default_type text/html;
    location / {
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_pass http://172.17.0.3;            # WordPress IP
    }
}
```

啟動 Nginx：
```bash
docker run -d --rm \
    -p 80:80 \
    -v `pwd`/wp.conf:/etc/nginx/conf.d/default.conf \
    nginx:alpine
```

#### 4.验证网站運行
- 瀏覽器访问 `http://127.0.0.1` 或虛擬機IP
-登入 MariaDB 验证 WordPress 已创建数据表

### 容器技术的局限
實戰後發現的問題：
- 需手动運行命令、人工確認狀態
- 多容器組合需人工干预（如检查 IP 地址）
- 网路模式只適合單機，多機負載均衡無法處理
- 增加應用數量時容器技术無法幫忙

這些問題指向 **容器编排（Container Orchestration）** 的需求，正是 Kubernetes 的核心功能。

## 重點摘要
- Registry 私有倉庫搭建簡單，只需拉取鏡像、映射端口、推送鏡像
- WordPress 部署需三容器協作：MariaDB（數據庫）→ WordPress（應用）→ Nginx（代理）
- 容器 IP 需用 `docker inspect` 查看，容器間靠 IP 通信
- 容器技術仍有局限，需容器编排技術（Kubernetes）解決

##關鍵字
Registry, WordPress, MariaDB, Container Orchestration, docker inspect