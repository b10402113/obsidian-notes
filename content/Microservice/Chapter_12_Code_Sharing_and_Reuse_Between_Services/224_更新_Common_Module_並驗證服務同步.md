# 更新 Common Module 並驗證服務同步

## 📝 課程概述

本單元處理的是日常開發中最頻繁發生的情境：當 Common Module 有新變更時，如何重新發布套件，並讓下游的 Auth Service 取得最新版本。我們也會實際進入 Kubernetes Pod 內部驗證容器是否真的執行了最新版本的程式碼。

---

## 核心觀念與實作解析

### 更新循環的完整流程

每當我們在 Common Module 中修改程式碼並希望其他服務看到這些變更時，必須依序執行以下步驟：

```bash
# 1. 在 Common Module 中修改程式碼
# ...

# 2. 執行 pub script（commit + version bump + build + publish）
cd common
npm run pub

# 3. 在 Consumer 服務中更新依賴版本
cd ../auth
npm update @sg-tickets/common

# 4. 如果是 Kubernetes 部署，需要重啟 Pod 以安裝新版本
kubectl delete pod <pod-name>
```

### 驗證版本同步的方法

理論上我們知道更新已經完成，但**如何確認 Container 真的執行了新版本**？老師示範了直接進入 Pod 內部查看 `node_modules` 的技巧：

```bash
# 取得所有 pods
kubectl get pods

# 進入特定 Pod 的 shell
kubectl exec -it <pod-name> -- /bin/sh

# 查看已安裝的 Common Module 版本
cd node_modules/@sg-tickets/common
cat package.json | grep version
```

如果顯示的 version 與我們剛發布的版本一致，代表更新確實生效了。

> **為什麼要刪除 Pod 而不是重啟？** 在 Kubernetes 中，刪除 Pod 後，Deployment 會自動根據更新後的 `package.json` 重新建立 Pod，並執行 `npm install` 安裝最新版本的 dependencies。這是確保所有 Containers 都執行一致版本的最乾淨方式。

---

### `files` 欄位的實際效果

在 Common Module 的 `package.json` 中有這個設定：

```json
"files": ["build/**/*"]
```

當其他服務執行 `npm install @sg-tickets/common` 時，**只有 `build/` 目錄會被下載**，`src/` 資料夾（TypeScript 原始碼）不會出現在 `node_modules` 中。這是刻意為之的設計：

- **保護智慧財產**：TS 原始碼不需要暴露給下游
- **減少安裝體積**：下游只需要編譯後的 JavaScript，不需要 TypeScript 工具鍊
- **強制相容性**：所有 consumer 都只能依賴經過編譯驗證的穩定輸出

---

### Auth Service 更新後的回歸測試

當我們完成所有 import 路徑的修正並確保 Common Module 正確安装後，需要做一個基本的 smoke test：

1. 啟動 `skaffold dev`
2. 開啟瀏覽器存取 `ticketing.dev`
3. 嘗試登入 / 註冊 / 登出流程

如果頁面正常渲染且認證流程運作如常，代表遷移完全成功。

---

### 為什麼要有 Common Module？

回顧這整個 Chapter 12 的意義：建立了 Common Module 之後，我們擁有了一套**可複用元件庫**。未來建立 Ticket Service、Orders Service 或任何新服務時，所有 authentication、error handling 與 request validation 的邏輯都已經準備好了，大幅降低啟動新服務的時間成本。

這也符合 Microservice 架構中「**每個 Service 自治**」的精神——共同關注點（cross-cutting concerns）被抽取到獨立的共享模組，而不是重複實作在每個服務中。

---

## 💡 重點摘要

- **更新 Common Module 的完整流程：修改程式碼 → `npm run pub` → 下游服務 `npm update` → 重啟 Pod。**
- **直接進入 Kubernetes Pod 內部檢查 `node_modules/@org/common/package.json` 的 version，是驗證部署是否正確的最可靠方式。**
- **`files: ["build/**/*"]` 確保只有編譯後的 JavaScript 會被 publish 和安裝，TS 原始碼不會外流。**
- **Common Module 大幅簡化了未來新增服務時的工作量——認證、錯誤處理、驗證邏輯無需重寫。**
- **`npm update` 會自動將套件升級到 `package.json` 中允許的最高版本，而不是強制的 latest。**

---

## 🔑 關鍵字

`npm run pub`, `npm update`, Kubernetes Pod, `files`, `node_modules`, Common Module, Smoke Test
