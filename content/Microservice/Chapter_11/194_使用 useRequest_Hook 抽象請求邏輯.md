# 使用 useRequest Hook 抽象請求邏輯

## 📝 課程概述

本單元將 Sign Up 表單中重複的**請求邏輯與錯誤處理**抽取成一個**自訂 Hook：`useRequest`**。這個 Hook 接收 URL、HTTP Method 與 Request Body，回傳一個 `doRequest` 函式與預先渲染好的錯誤 JSX，大幅簡化未來所有表單的實作。

## 核心觀念與實作解析

### 為何需要抽取 Hook？

本專案未來會有 Sign In、Sign Up、Create Ticket、Edit Ticket 等多個表單，每個表單幾乎都需要：

1. 發送非同步請求
2. 成功後做頁面跳轉（`router.push`）
3. 失敗時渲染錯誤訊息

如果把這些邏輯直接寫在每個 Component 裡，未來改動時要改 N 處。**抽取成 Hook 是正確的抽象方向。**

### `useRequest` 的介面設計

```javascript
// 輸入
const { doRequest, errors } = useRequest({
  url: '/api/users/signup',
  method: 'post',
  body: { email, password },
  onSuccess: () => router.push('/'),  // 成功後跳轉
});

// 使用
await doRequest();          // 執行請求
{errors}                    // 直接渲染錯誤 JSX
```

### Hook 內部實作

```javascript
export const useRequest = ({ url, method, body, onSuccess }) => {
  const [errors, setErrors] = useState(null);

  const doRequest = async () => {
    try {
      setErrors(null);  // 新請求開始時清除舊錯誤
      const response = await axios[method](url, body);
      if (onSuccess) onSuccess(response.data);
    } catch (error) {
      setErrors(
        <div className="alert alert-danger">
          <h4>Oops...</h4>
          <ul className="my-0">
            {error.response.data.errors.map((err) => (
              <li key={err.message}>{err.message}</li>
            ))}
          </ul>
        </div>
      );
    }
  };

  return { doRequest, errors };
};
```

### 成功後重新導向

在 `useRequest` 中加入 `onSuccess`  Callback 參數，確保請求成功後才執行跳轉：

```javascript
if (onSuccess) onSuccess(response.data);
```

這樣即使 `doRequest` 在 Component 的 `onSubmit` 中使用 `await`，錯誤也會被 Hook 內部消化，不會繼續執行 `router.push`——**不需要再手動 re-throw error**。

### 登出按鈕與成功跳轉的差異

Sign Up / Sign In 成功後的跳轉是「引導頁面轉換」，但**不能從 `getInitialProps` 內部發出**，因為伺服器端無法處理 Cookie 操作。Sign Out 的請求必須由 Browser Component 發出，這點在後續章節會再深入說明。

---

## 💡 重點摘要

- **抽取重複邏輯到 Hook 是 React 的核心最佳實踐，能讓程式碼 DRY 且易於維護。**
- **`onSuccess` Callback 讓 Hook 的呼叫端自行決定成功後的行為，兼顧彈性與封裝。**
- **`setErrors(null)` 放在 `try` 區塊最開頭，確保新請求開始時自動清除上一次的錯誤訊息。**

## 🔑 關鍵字

useRequest, 自訂 Hook, DRY, onSuccess Callback, axios[method], 請求抽象
