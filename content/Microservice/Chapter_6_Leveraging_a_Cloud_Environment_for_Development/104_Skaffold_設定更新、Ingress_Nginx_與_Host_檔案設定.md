# Skaffold 設定更新、Ingress Nginx 與 Host 檔案設定

## 📝 課程概述

本單元完成 Skaffold 設定的最後調整（停用本地建置、更新 Deployment Image）、在 GKE 叢集上重新部署 **Ingress Nginx**，並更新本機的 `hosts` 檔案，將 `ticketing.dev` 的流量導向 Google Cloud Load Balancer 的 IP。

---

## 核心觀念與實作解析

### 停用 Skaffold 的本地建置設定

在 `skaffold.yaml` 中，`build` 區塊裡的 `local` 設定（`push: false`）和 `googleCloudBuild` 設定**只能存在一個**。因此我們需要將 `local` 區塊**註釋掉**：

```yaml
build:
  # local:
  #   push: false        # 註釋掉，不再使用
  googleCloudBuild:
    projectId: <your-project-id>
```

---

### 更新 Deployment 檔案中的 Image 名稱

Deployment 檔案（例如 `infra/k8s/auth-depl.yaml`）中的 `spec.template.spec.containers[].image` 欄位，原本指向本地 Docker Hub 的 Image 名稱，例如：

```yaml
image: <your-docker-id>/auth
```

現在需要**同步更新為 GCR 的 Image URL**：

```yaml
image: us.gcr.io/<projectId>/auth
```

> **這是一個容易被忽略的步驟**：如果你忘記更新 Deployment 檔案，Pod 啟動時仍然會去 Docker Hub 找舊 Image，導致部署失敗。

---

### 在 GKE 叢集上重新部署 Ingress Nginx

Ingress Nginx 的設定**不會**隨著叢集遷移而自動帶過去，必須在新的 GKE 叢集上重新執行初始化命令。

#### 步驟一：執行 Mandatory 命令

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud/deploy.yaml
```

這會在 GKE 叢集內建立 **Ingress Controller**——一個承載 Nginx 路由規則的 Pod，負責根據 ingress 資源的設定將流量分配到各 Service。

#### 步驟二：執行 Google Cloud 專屬命令

在 Ingress Nginx 官方文件的 Google Cloud 頁面中，會有一個額外的 GCP 專屬命令：

```bash
kubectl apply -f <GCP-特有設定-URL>
```

這個額外設定會在 GCP 上建立一個 **Cloud Load Balancer**——這是 GCP 本身的網路資源，**運行在叢集外部**。Load Balancer 負責接收外部流量並轉發到 Ingress Controller。

> **理解 Load Balancer 的角色非常重要**：當我們在瀏覽器輸入 `ticketing.dev` 時，瀏覽器並非直接連線到叢集內的 Pod，而是先連線到這個 Load Balancer，再由 Load Balancer 轉發給 Ingress Nginx，最後才路由到對應的 Service。

---

### 查詢 Load Balancer 的 IP 位址

Ingress Nginx 部署完成後，會自動在 GCP 上建立一個 Load Balancer。你可以在 Google Cloud Console 中查詢其 IP：

1. 左側選單 → **Network Services** → **Load Balancing**
2. 你會看到一個自動建立的 Load Balancer，目標是你的 auth Service
3. 點進去即可看到其 **External IP**（格式如 `34.x.x.x`）

---

### 更新 hosts 檔案

將 `ticketing.dev` 指向 Load Balancer 的 IP，這樣瀏覽器的請求才會被送到 GCP 的遠端叢集。

**macOS / Linux** (`/etc/hosts`)：

```
<Load-Balancer-IP>  ticketing.dev
```

**Windows** (`C:\Windows\System32\drivers\etc\hosts`)：

```
<Load-Balancer-IP>  ticketing.dev
```

編輯 `hosts` 檔案需要管理員/root 權限。

> **這裡有一個常見陷阱**：如果你之前已經在 hosts 檔案中設定過 `ticketing.dev`，記得**更新現有那一行**，而非新增一行，否則會造成衝突。

---

## 💡 重點摘要

- **Skaffold 的 `local` 和 `googleCloudBuild` 設定互斥，啟用 Cloud Build 時必須註釋掉 `local` 區塊。**
- **Deployment 檔案中的 Image URL 必須同步更新為 GCR 格式，否則 Pod 會嘗試拉取 Docker Hub 上不存在的 Image。**
- **Ingress Nginx 的部署分兩階段：先建立 Ingress Controller（叢集內部），再執行 GCP 專屬命令建立 Load Balancer（叢集外部）。**
- **Load Balancer 是 GCP 提供的外部服務，它接收所有外部流量並轉發給叢集內的 Ingress Nginx。**
- **更新 `hosts` 檔案後，`ticketing.dev` 的流量將從本地迴路改為導向 GCP Load Balancer。**

---

## 🔑 關鍵字

hosts, Ingress Nginx, Cloud Load Balancer, GCP, Ingress Controller, GCR, Deployment, Image
