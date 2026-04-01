# 緩解 Option 2 安全問題的可行策略

## 📝 課程概述

本單元介紹兩種**繞過 Option 2 安全漏洞**的可行策略，雖然課程最終不會實作這些方案，但理解它們有助於學員在未來面對類似需求時有解決思路。我們將探討 Token 過期機制與 Event Bus 通知兩種方向的優劣。

---

## 核心觀念與實作解析

### 策略一：JWT 強制過期 + Refresh Token 機制

**核心思路**：不要讓 JWT 永久有效，強制規定 JWT 只能使用一個**短暫時段**（例如 15 分鐘）。過期後必須重新向 Auth Service 換發新 JWT。

```
使用者攜帶 30 分鐘前取得的 JWT 發出請求
→ Order Service 檢查：「這 JWT 已過期，拒絕」
→ 使用者（或客戶端自動）向 Auth Service 申請 Refresh Token，換發新 JWT
→ 這次 Auth Service 可以順便檢查：「此使用者是否被封鎖？」
→ 若已封鎖，拒絕換發
```

**策略一的好處**：
- Auth Service 在每次 Refresh 時都有機會介入、檢查最新狀態
- 封鎖生效延遲最長為一個 Token 生命週期（例如 15 分鐘）

**策略一的代價**：
- 仍有時間窗口（取決於 Token 有效期）
- 客戶端需要實作 Refresh 邏輯

---

### 策略二：封鎖事件廣播（最嚴格方案）

**核心思路**：管理員封鎖使用者時，Auth Service 除了更新資料庫，還**同步發出一個 `UserBanned` 事件**到 Event Bus。所有服務監聽此事件，並將被封鎖的使用者 ID 存入**短暫記憶體快取**（TTL 等於 JWT 有效期）。

```
Auth Service: UPDATE database → EMIT UserBanned event → Event Bus
                                                    ↓
各 Service 收到事件 → 寫入本地記憶體快取（TTL = JWT 有效時間）
                                    ↓
Order Service 驗證 JWT 前 → 先查詢快取：「這個人是否被封鎖？」
→ 若是，直接拒絕，完全繞過 JWT 驗證
```

**為什麼快取只需要短暫存在？**

因為 JWT 本身有過期時間。當 JWT 自然過期後，即使快取已清除，系統也會拒絕使用者的舊請求。快取只是為了填補「JWT 尚未過期，但使用者已被封鎖」這段時間窗口。

**策略二的好處**：
- 封鎖效果幾乎**立即生效**（取決於 Event Bus 的傳遞延遲）

**策略二的代價**：
- 每個服務都需要實作監聽邏輯、快取管理邏輯
- 增加架構複雜度（但仍可透過 shared library 封裝）

---

## 💡 重點摘要

- **JWT 強制過期 + Refresh 機制能大幅縮短封鎖窗口，但仍有殘餘延遲（等於 Token 生命週期）。**
- **Event Bus 廣播 + 短期快取是最嚴格的方案，可以近乎即時地讓所有服務知道使用者被封鎖。**
- **兩種策略都會增加實作複雜度，課程選擇不實作它們，繼續使用基本 Option 2，以保持架構簡潔。**

---

## 🔑 關鍵字

Refresh Token, TTL, Event Bus, UserBanned, JWT Expiration, Short-lived Cache
