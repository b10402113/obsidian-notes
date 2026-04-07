# GitHub Actions 基礎範例

## 📝 課程概述

本單元開始進入實戰範例，講師將透過 10 個層層疊加的範例，帶領學員成為「Workflow Jedi」。第一個範例涵蓋最基本的 GitHub Actions 工作流：登入 Registry 並建置 Image。

---

## 核心觀念與實作解析

### GitHub Actions 核心概念

#### 層級結構

```
Workflow（工作流）→ Job（作業）→ Step（步驟）
```

- **Workflow**：一個 YAML 檔案就是一個 Workflow
- **Job**：一個 Workflow 可以包含多個 Job
- **Step**：一個 Job 可以包含多個 Step

#### 觸發事件

本系列主要使用兩個事件：

| 事件 | 觸發時機 |
| --- | --- |
| `pull_request` | 當 PR 建立或有新 commit 推送到 PR |
| `push` | 當程式碼推送到指定分支（如 main） |

```yaml
on:
  pull_request:
  push:
    branches: [main]
```

---

### 基礎工作流範例

#### Step 的定義格式

每個 Step 使用 GitHub Action 的標準格式：

```yaml
- name: Step Name
  uses: organization/action-name@version
```

例如：

```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

#### 建置與推送 Image

```yaml
- name: Build and push
  uses: docker/build-push-action@v4
  with:
    context: .
    push: ${{ github.event_name != 'pull_request' }}
    tags: user/image:latest,user/image:v1
```

#### 關鍵邏輯：條件推送

```yaml
push: ${{ github.event_name != 'pull_request' }}
```

這行程式碼的意思是：

- **PR 階段**：只建置 Image，**不推送**到 Registry
- **合併後（push to main）**：建置並推送 Image 到 Registry

> **Why 這樣設計？** 避免在 PR 階段產生大量臨時測試 Image，污染 Registry。建置本身就是一種測試——如果 Dockerfile 有問題，建置會失敗。

---

### 官方 Docker Actions

Docker 官方提供了六個 GitHub Actions，本系列會用到其中四到五個：

| Action | 用途 |
| --- | --- |
| `docker/login-action` | 登入 Registry |
| `docker/build-push-action` | 建置與推送 Image |
| `docker/metadata-action` | 產生標籤與中繼資料 |
| `docker/setup-buildx-action` | 設定 BuildX |
| `docker/setup-qemu-action` | 設定跨平台建置 |

---

## 💡 重點摘要

- **GitHub Actions 的層級結構為：Workflow（檔案）→ Job → Step。**
- **本系列主要使用 pull_request 和 push 兩個觸發事件。**
- **基礎工作流包含登入 Registry 和建置 Image，PR 階段不推送，合併後才推送。**
- **Docker 官方提供六個 GitHub Actions，本系列會用到四到五個。**

---

## 🔑 關鍵字

GitHub Actions, Workflow, Job, Step, docker/login-action
