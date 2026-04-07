# Shell Form 與 Exec Form

## 📝 課程概述

本單元深入探討 Dockerfile 中指令的兩種語法形式：**Shell Form** 與 **Exec Form**。理解這兩種形式的差異，對於正確使用 `ENTRYPOINT` 與 `CMD` 至關重要，特別是在處理 Signal 傳遞與程序管理的場景。

---

## 核心觀念與實作解析

### 兩種語法形式

Dockerfile 中有三個指令可以執行程式：`RUN`、`CMD`、`ENTRYPOINT`。它們支援兩種語法：

#### Shell Form

```dockerfile
RUN apt-get update && apt-get install -y curl
CMD nginx -g "daemon off;"
ENTRYPOINT python app.py
```

特徵：**只是字串**，Docker 會在 Shell 中執行。

#### Exec Form

```dockerfile
RUN ["apt-get", "update"]
CMD ["nginx", "-g", "daemon off;"]
ENTRYPOINT ["python", "app.py"]
```

特徵：**JSON Array 語法**，Docker 直接執行，不經過 Shell。

---

### Shell Form 的運作

Shell Form 會在指令前加上 `sh -c`：

```dockerfile
CMD nginx -g "daemon off;"
# 實際執行：sh -c "nginx -g 'daemon off;'"
```

**Shell Form 的優點：**
- 可使用 Shell 功能：變數替換、`&&` 串接、Pipe、重導向
- 語法較簡潔

**Shell Form 的缺點：**
- 程式不是 PID 1，而是 Shell 的子程序
- Signal（如 SIGTERM）可能無法正確傳遞

---

### Exec Form 的運作

Exec Form 告訴 Docker 直接執行指令，不經過 Shell：

```dockerfile
CMD ["nginx", "-g", "daemon off;"]
# 實際執行：nginx -g "daemon off;"（直接執行）
```

**Exec Form 的優點：**
- 程式直接成為 PID 1
- 可正確接收 Signal（SIGTERM、SIGINT）
- 適合長時間執行的程序

**Exec Form 的缺點：**
- 無法使用 Shell 功能（變數替換、Pipe 等）
- 語法較繁瑣

---

### SHELL 指令

預設 Shell 是 `sh`，但可以變更：

```dockerfile
# Linux 使用 Bash
SHELL ["/bin/bash", "-c"]

# Windows 使用 PowerShell
SHELL ["powershell", "-command"]
```

---

### 何時使用哪種形式？

老師提供簡單的規則：

| 指令 | 建議形式 | 原因 |
|------|---------|------|
| `RUN` | **Shell Form** | 需要 Shell 功能（`&&`、Pipe 等） |
| `ENTRYPOINT` | **Exec Form** | 確保 PID 1 正確 |
| `CMD` | **Exec Form**（預設） | 確保 PID 1 正確 |
| `ENTRYPOINT + CMD` | **都用 Exec Form** | 避免混用造成問題 |

**重要提醒：**

> **不要混用 Exec Form 與 Shell Form。** 當 `ENTRYPOINT` 和 `CMD` 一起使用時，兩者都應使用 Exec Form。

---

### 為什麼 PID 1 很重要？

PID 1 是 Container 的第一個程序，負責接收 Signal：

- `docker stop` 發送 SIGTERM
- `Ctrl+C` 發送 SIGINT

如果程序不是 PID 1（因為 Shell Form），Signal 會發給 Shell 而非你的應用程式，導致：
- 無法優雅關閉（Graceful Shutdown）
- Docker 被迫強制終止程序（等待 10 秒後 kill）
- 可能造成資料損壞

> **這就是為什麼官方 Image 的 `CMD` 都使用 Exec Form。**

---

## 💡 重點摘要

- **Shell Form 是字串語法，會在 Shell 中執行；Exec Form 是 JSON Array，直接執行。**
- **`RUN` 建議用 Shell Form，方便使用 `&&`、Pipe 等 Shell 功能。**
- **`ENTRYPOINT` 和 `CMD` 建議用 Exec Form，確保程序是 PID 1，能正確接收 Signal。**
- **混用 Exec Form 與 Shell Form 會造成問題，應避免。**
- **PID 1 負責接收 Signal，非 PID 1 的程序可能無法優雅關閉。**

---

## 🔑 關鍵字

Shell Form, Exec Form, PID 1, Signal, SHELL, JSON Array
