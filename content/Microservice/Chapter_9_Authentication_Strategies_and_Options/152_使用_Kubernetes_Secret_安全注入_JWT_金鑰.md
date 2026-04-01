# 使用 Kubernetes Secret 安全注入 JWT 金鑰

## 📝 課程概述

本單元介紹如何利用 **Kubernetes Secret** 來安全儲存 JWT Signing Key，並透過 Deployment 設定將其注入為各 Service 的**環境變數**。我們也會在 Node.js 程式碼啟動時檢查環境變數是否存在，確保 Deployment 設定錯誤能夠被及時發現。

---

## 核心觀念與實作解析

### 建立 Kubernetes Secret

```bash
kubectl create secret generic jwt-secret --from-literal=JWT_KEY=my-super-secret-key
```

- `generic`：代表這是一個**通用（all-purpose）Secret**，用於儲存任意 Key-Value 資料
- `jwt-secret`：Secret 的名稱，用於之後在 Deployment 中引用
- `--from-literal`：直接以命令列參數形式提供 Key-Value，而非從檔案讀取

> **注意**：`kubectl create secret` 是**命令式（Imperative）**操作——直接執行命令建立物件，而非寫 YAML 檔案。這是刻意選擇：將 Secret 的實際值寫進 YAML 檔案並放進 Git，會讓金鑰暴露在版本控制中。

---

### 將 Secret 注入為環境變數

在 Deployment YAML 的 container spec 中新增：

```yaml
env:
  - name: JWT_KEY          # 容器內的環境變數名稱
    valueFrom:
      secretKeyRef:
        name: jwt-secret   # 引用上面的 Secret 名稱
        key: JWT_KEY       # Secret 內的 Key 名稱
```

**常見錯誤**：若 Secret 名稱或 Key 名稱拼錯，Kubernetes **不會啟動 Pod**，並顯示 `CreateContainerConfigError`。除錯方法：`kubectl describe pod <pod-name>` 即可看到詳細錯誤資訊。

---

### 在 Node.js 啟動時檢查環境變數

```typescript
// index.ts - start() 函數最開頭
if (!process.env.JWT_KEY) {
  throw new Error('JWT_KEY must be defined!');
}
```

**為什麼要放在 `start()` 而非各 Route Handler 中？**

- 若在 Route Handler 中檢查，錯誤只會在使用者**第一次訪問需要驗證的路由時**才會拋出
- 在 `start()` 中檢查，確保**程式啟動時**立即發現問題，避免在正式環境中帶病運行

---

### 程式碼中讀取環境變數

```typescript
const token = jwt.sign(payload, process.env.JWT_KEY!);
//                                    ↑
//  TypeScript 認為可能是 undefined，所以需要 ! 斷言
```

使用 `!` 斷言（Non-null assertion）告訴 TypeScript：「我已經在 `start()` 中檢查過了，這裡不可能是 undefined。」

---

## 💡 重點摘要

- **`kubectl create secret` 是 imperative 命令，每次建立新叢集時都需重新執行，建議將命令本身備份到安全位置。**
- **Deployment 中的 `secretKeyRef` 讓 Secret 值在 Pod 啟動時才注入，Secret 本身不會暴露在 YAML 檔案中。**
- **在 `start()` 函數最開頭檢查環境變數，能在部署當下立即發現設定錯誤，而非等到使用者請求時才失敗。**

---

## 🔑 關鍵字

Kubernetes Secret, secretKeyRef, Environment Variable, kubectl create secret, JWT_KEY
