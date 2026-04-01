# Common Module 專案初始化與建置流程

## 📝 課程概述

本單元實作建立 Common Module 的完整流程：從初始化專案、撰寫 TypeScript、配置 `tsconfig.json`、到執行首次建置並成功發布到 NPM Organization。我們將解決「用 TypeScript 寫作，但發布 JavaScript」這個關鍵技術決策。

---

## 核心觀念與實作解析

### 為什麼「寫 TypeScript，發布 JavaScript」？

這是一個至關重要的設計決策，原因有兩個：

1. **版本相容性**：未來某個服務可能使用不同版本的 TypeScript，如果 Common Module 也要求特定版本，就會造成衝突。
2. **語言中性**：我們的 microservices 目前用 TypeScript 開發，但不代表未來所有服務都會用 TypeScript——有可能某個新服務是用純 JavaScript 寫的。

**解決方案**：Common Module 用 TypeScript 開發以獲得型別安全的好處，但在 publish 之前，先用 TypeScript Compiler（`tsc`）將其編譯成純 JavaScript。這樣任何語言、任何 TypeScript 版本的服務都可以安心使用。

---

### 初始化專案的步驟

```bash
mkdir common && cd common
npm init -y
```

此時產生的 `package.json`，`name` 欄位需要修改為 Organization 的格式：

```json
{
  "name": "@sg-tickets/common"
}
```

---

### 安裝建置所需的開發依賴

```bash
npm install --save-dev typescript tsx
npm install --save-dev @types/express @types/cookie-session @types/jsonwebtoken @types/express-validator
npm install express cookie-session jsonwebtoken express-validator
```

**為什麼要安裝這些 runtime dependencies？**

因為當我們把 `middleware` 與 `errors` 程式碼移入 Common Module 後，這些檔案內部引用了 `express`、`cookie-session`、`jsonwebtoken` 等套件。如果不安裝，建置時 TypeScript 會回報「模組找不到」的錯誤。

---

### `tsconfig.json` 的關鍵設定

```json
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "declaration": true,
    "outDir": "./build"
  }
}
```

- **`declaration: true`**：這個設定會在編譯時同時產生 `.d.ts` 型別定義檔。即使 Common Module 發布的是 JavaScript，使用 TypeScript 的服務仍然可以獲得完整的型別推斷能力。
- **`outDir: "./build"`**：所有編譯產出的 JavaScript 與 `.d.ts` 檔案都會放到 `build/` 目錄，而不是覆蓋 source code。

---

### package.json 中的四個關鍵欄位

```json
{
  "name": "@sg-tickets/common",
  "main": "build/index.js",
  "types": "build/index.d.ts",
  "files": [
    "build/**/*"
  ]
}
```

- **`main`**：當其他服務 `import xxx from '@sg-tickets/common'` 時，Node.js 會去讀取這個路徑的檔案。
- **`types`**：TypeScript compiler 會參考這個檔案來解析型別資訊。
- **`files`**：明確告訴 NPM 發布時要包含哪些檔案（只包含 `build/` 目錄，不包含 source TypeScript 檔案）。

---

### 建置與發布流程

每次更新 Common Module 並發布，需要依序執行以下命令：

```bash
npm run build       # 執行 tsc，將 TypeScript → JavaScript 到 build/
npm version patch   # 自動遞增 version (1.0.0 → 1.0.1)
npm publish         # 發布到 NPM（需確認有 --access public）
```

### 自動化 script：`pub`

因為在這個課程中我們會頻繁更新 Common Module，老師設計了一個 `pub` script 來一鍵完成所有步驟：

```json
"scripts": {
  "clean": "rimraf build",
  "build": "npm run clean && tsc",
  "pub": "git add . && git commit -m \"update\" && npm version patch && npm run build && npm publish"
}
```

> ⚠️ **重要提醒**：這個 `pub` script 只適合在本課程中用來簡化操作。現實專案中，你不會想用這麼暴力的方式（每次都用同一個 generic commit message、強制只更新 patch version），而是會謹慎地選擇 version bump 的粒度（patch / minor / major）。

---

## 💡 重點摘要

- **Common Module 用 TypeScript 開發是為了型別安全，發布 JavaScript 是為了最大相容性。**
- **`declaration: true` 讓 TypeScript 編譯時同時輸出 `.d.ts` 檔案，確保使用者的 IDE 仍能獲得完整的型別提示。**
- **`main`、`types`、`files` 三個欄位是 NPM package 能被正確消費的關鍵配置。**
- **`files` 陣列只指定 `build/**/*`，可以避免 source code（TypeScript）被意外發布到 NPM。**
- **`npm version patch` 會自動更新 `package.json` 的 version 欄位，這是 semantic versioning 的標準操作。**

---

## 🔑 關鍵字

TypeScript Compiler, `declaration`, `outDir`, Semantic Versioning, `npm version patch`, `build/`, `.d.ts`
