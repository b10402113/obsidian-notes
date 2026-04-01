# 使用 SuperTest 撰寫第一個整合測試

## 📝 課程概述

本節正式開始撰寫測試程式碼。我們會建立第一個測試檔案，確認當發送有效資料到 `POST /api/users/signup` 時，能正確得到 201 狀態碼。同時也會處理一個實務上常見的問題：`JWT_KEY` 環境變數在測試環境中不存在，導致請求失敗。

## 核心觀念與實作解析

### Jest 測試檔案的命名與位置慣例

Jest 有自己的慣例：若要測試某個檔案，**在同目錄下建立 `__test__` 資料夾**，並以原始檔案名稱加上 `.test.ts` 作為測試檔名。

```
src/routes/
├── signup.ts
└── __test__/
    └── signup.test.ts   ← 測試檔案
```

### 第一個測試：成功註冊回傳 201

```typescript
import request from 'supertest';
import { app } from '../../app';

it('returns a 201 on successful signup', async () => {
  return request(app)
    .post('/api/users/signup')
    .send({ email: 'test@test.com', password: 'password' })
    .expect(201);
});
```

SuperTest 的使用方式非常直覺：
1. `request(app)`：傳入我們的 Express App
2. `.post('/api/users/signup')`：指定 HTTP Method 與路徑
3. `.send({ ... })`：攜帶 Request Body
4. `.expect(201)`：斷言預期的 HTTP 狀態碼

### 常見陷阱：`JWT_KEY` 環境變數

執行測試時可能會看到 `400 Bad Request`，原因是：`signup.ts` 內部在建立 JWT 時，需要讀取 `process.env.JWT_KEY`，但在測試環境中這個變數從未被設定（它只在 Kubernetes Pod 的環境變數中定義）。

解法：在 `setup.ts` 的 `beforeAll` 中直接設定：

```typescript
process.env.JWT_KEY = 'test-jwt-key';
```

這不是最優雅的解法（正式環境應透過 Kubernetes Secret 或 Vault 等機制管理），但對測試環境而言完全足夠。

### Jest 的 TypeScript 注意事項

Jest 預設並不原生支援 TypeScript。`ts-jest` 提供了轉譯層，但偶爾會出現「改了 TS 檔但 Jest 仍使用舊版程式碼」的問題。若測試結果不如預期，**第一件事：重新啟動 Jest（`Ctrl+C` 後再跑一次 `npm run test`）**，這通常能解決問題。

### 測試函式加上 `async` 的理由

雖然第一個測試中 `await` 只出現在 `request()` 鏈中，但我們仍將測試函式標記為 `async`。這是為了未來在單一測試內需要執行多個請求時，能直接使用 `await`，而不需要事後再加回來。

## 💡 重點摘要

- Jest 的 `__test__` 資料夾命名慣例是標準做法，遵守它讓專案結構一致
- SuperTest 的鏈式 API 設計極為直覺，是測試 Express Handler 的首選工具
- `JWT_KEY` 環境變數在測試環境中需手動設定，Production 環境則由 Kubernetes 注入
- 若遇到「改了 code 但測試沒變」的詭異行為，先重啟 Jest 再說
- 將測試函式預設標為 `async`，為未來多步驟測試預做準備

## 🔑 關鍵字

SuperTest, request(), .expect(), JWT_KEY, __test__, ts-jest
