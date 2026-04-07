# PHP Apache 與 Alpine Dockerfile 進階審查

## 📝 課程概述

本單元延續 PHP Dockerfile 審查系列，涵蓋兩個案例：一是使用 PHP Apache 的 Drupal 專案，二是使用 PHP-FPM Alpine 的 Production Image。老師針對版本管理、Alpine 的潛在問題、Healthcheck、Supervisor 等議題提供專業建議。

---

## 核心觀念與實作解析

### 案例一：Drupal PHP Apache Dockerfile

#### 整體評價

這是一個結構完整的 Dockerfile，包含 PHP extension 安裝、Composer、Drupal 設定等。但有幾個優化建議：

#### 建議一：固定 PHP Extension 版本

```dockerfile
# 原始寫法（風險）
RUN docker-php-ext-install mysqli pdo_mysql

# 建議寫法
RUN apt-get install -y \
    libpq-dev=13.x.x \
    && docker-php-ext-install mysqli pdo_mysql
```

老師再次強調：**他曾在 Production 遇到 PHP driver（Postgres/Mongo）更新後直接崩潰的慘痛經驗**。

#### 建議二：環境變數用於彈性設定

原始 Dockerfile 硬編碼了 PHP 的 memory limit 等設定：

```dockerfile
RUN echo "memory_limit = 256M" >> /usr/local/etc/php/php.ini
```

建議改用環境變數，讓 dev 與 production 可以有不同設定：

```dockerfile
ENV PHP_MEMORY_LIMIT=256M
# 在 php.ini 中引用 ${PHP_MEMORY_LIMIT}
```

#### 建議三：版本變數集中管理

將所有版本號集中在 Dockerfile 開頭：

```dockerfile
ARG PHP_VERSION=8.1
ARG DRUPAL_VERSION=9.4
ARG COMPOSER_VERSION=2.4

FROM php:${PHP_VERSION}-apache
```

這樣可以一眼看出專案使用的所有版本，維護起來更方便。

---

### 案例二：PHP-FPM Alpine Production Image

#### Alpine 的版本固定問題

這個案例暴露了 Alpine 的另一個問題：

> **在 Alpine 中，APK packages 無法正確 pin 版本！**

當 Alpine 版本更新時，舊版本的 packages 可能直接從 repository 消失，這意味著：

- 你無法保證 reproducible builds
- 可能某天 build 突然失敗

#### Alpine vs Debian Slim 的選擇

老師分享了他的觀點：

| 考量 | Alpine | Debian Slim |
|------|--------|-------------|
| CVE Scanning | 困難 | 容易 |
| 安全性 | 較差（近期問題多） | 較佳 |
| 大小差異 | 小 | 相差不多（~50MB） |
| 版本固定 | 無法 | 可以 |

> 老師建議：如果無法 pin 版本，且你曾因版本問題遭遇 Production outage，應該考慮使用 Debian Slim。

#### 優點：Healthcheck 設定

```dockerfile
HEALTHCHECK CMD curl -f http://localhost/status || exit 1
```

這是最佳實踐！

- **Docker Swarm** 會自動利用 Healthcheck
- **Kubernetes** 可以搭配 livenessProbe / readinessProbe 使用

#### Supervisor 作為 Init Process

這個 Dockerfile 使用 Supervisor 來管理多個 Process：

```dockerfile
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
```

這是正確的做法，因為：

- 需要同時執行 PHP-FPM、Nginx、Cron
- Supervisor 可以作為 PID 1，正確處理 signals
- 避免 zombie processes

#### Stop Signal 的使用

```dockerfile
STOPSIGNAL SIGTERM
```

這是一個少見但正確的設定，確保 Container 停止時能優雅關閉連線。

---

### Multi-stage Builds 的建議

老師建議這類複雜的 Dockerfile 可以考慮 Multi-stage：

```dockerfile
# Build Stage - 安裝 build dependencies
FROM php:fpm AS builder
# 安裝 composer、build tools...

# Production Stage - 只保留執行需要的
FROM php:fpm AS production
COPY --from=builder /app /app
# 不包含 build dependencies
```

這樣可以：

1. 減少最終 Image 的 attack surface
2. 移除不需要的 build dependencies
3. 讓 Image 更精簡

---

## 💡 重點摘要

- **固定所有 PHP extensions 與 libraries 的版本，避免因自動更新導致 Production 問題。**
- **Alpine 的 APK packages 無法正確 pin 版本，這是選擇 Base Image 時的重要考量。**
- **Healthcheck 應該寫在 Dockerfile 中，讓 Swarm/Kubernetes 能自動監控容器健康狀態。**
- **多 Process 容器應使用 Supervisor 作為 init process，避免 zombie processes。**
- **Multi-stage Builds 可以分離 build 與 production dependencies，減少 Image 攻擊面。**

---

## 🔑 關鍵字

Alpine, Healthcheck, Supervisor, Multi-stage, STOPSIGNAL
