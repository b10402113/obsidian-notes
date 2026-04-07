# Database 升級實作：Named Volumes 應用

## 📝 課程概述

本單元透過一個真實場景——**Database 版本升級**——來驗證 Named Volumes 的實際應用。我們將學習如何在遵循「不修改 Container，而是替換 Container」的最佳實踐下，安全地完成 Database 的 patch 版本升級，同時確保資料完整保留。

---

## 核心觀念與實作解析

### 場景：為什麼 Database 升級是個問題？

想像一個情境：你的 PostgreSQL 需要從 9.6.1 升級到 9.6.2（patch 版本更新，可能是安全性修正或 bug fix）。

**傳統做法**：在 Server 上直接執行 package manager 升級

**Container 的最佳實踐**：不修改 Container 內的應用，而是用新版本 Image 重建 Container

> 問題來了：如何在替換 Container 的同時，保留 Database 內的所有資料？這正是 Named Volumes 發揮作用的時刻。

---

### 實作步驟：PostgreSQL 版本升級

#### 步驟 1：查詢 Docker Hub 取得 Volume 路徑

前往 Docker Hub 的官方 PostgreSQL Repository，找到目標版本（9.6.1）的 Dockerfile。

關鍵資訊：
```dockerfile
VOLUME /var/lib/postgresql/data
```

> 這就是 PostgreSQL 存放資料的位置，我們的 Named Volume 需要指向這裡。

---

#### 步驟 2：啟動舊版本 Container

```bash
docker container run -d --name psql -v psql:/var/lib/postgresql/data postgres:9.6.1
```

指令解析：
- `-d`：背景執行
- `--name psql`：Container 名稱
- `-v psql:/var/lib/postgresql/data`：Named Volume，名稱為 `psql`

---

#### 步驟 3：觀察初始化 Log

```bash
docker container logs -f psql
```

第一次啟動 Database Container 時，它會執行大量初始化工作：
- 建立 admin user
- 建立預設 database
- 設定權限等

> 當 Log 出現 `database system is ready to accept connections`，表示初始化完成。按 `Ctrl+C` 離開。

---

#### 步驟 4：停止舊 Container

```bash
docker container stop psql
```

---

#### 步驟 5：啟動新版本 Container（使用相同 Volume）

```bash
docker container run -d --name psql2 -v psql:/var/lib/postgresql/data postgres:9.6.2
```

關鍵點：
- **Volume 名稱必須相同**（`psql`），這樣才能存取舊 Container 的資料
- Container 名稱必須不同（`psql2`），因為 `psql` 還存在（只是停止）
- 確保舊 Container 已停止——兩個 Container 不能同時存取同一份資料

---

#### 步驟 6：驗證升級成功

```bash
docker container logs psql2
```

**關鍵觀察**：這次的 Log 會非常短——只有幾行，因為：
- Database 已經存在，不需要重新初始化
- 直接使用 Named Volume 內的現有資料
- 只要看到 `database system is ready to accept connections` 就成功了

```bash
# 確認只有一個 Named Volume
docker volume ls

# 確認兩個 Container 狀態
docker container ps -a
```

---

### 重要提醒：這個方法僅適用於 Patch 版本升級

**為什麼？**

大多數 Database 系統**不會自動進行 major version 升級**，因為：
- Major 升級通常涉及 Schema 變更
- 需要執行特定的 migration 工具

以 PostgreSQL 為例，從 9.x 升級到 10.x 需要 `pg_upgrade` 工具。

> 本單元介紹的方法適用於 **patch version（如 9.6.1 → 9.6.2）**，不涉及 Schema 變更或重大資料結構調整。

---

### 作業練習重點回顧

這個作業測試了以下能力：

1. **查閱官方 Dockerfile**：學會從 Docker Hub 找到 Volume 路徑
2. **Named Volume 語法**：`-v <name>:<container_path>`
3. **理解 Volume 生命週期**：Volume 與 Container 獨立存在
4. **Log 觀察技巧**：透過 Log 判斷是否使用現有資料
5. **版本升級流程**：停止舊 Container → 啟動新 Container（使用相同 Volume）

---

## 💡 重點摘要

- **Database 升級的最佳實踐是用新版本 Image 重建 Container，而非在 Container 內升級。**
- **Named Volume 讓資料與 Container 生命週期解耦，是新舊 Container 共用資料的關鍵。**
- **第一次啟動 Database Container 時 Log 會很長（初始化），第二次使用現有 Volume 時 Log 會很短。**
- **此方法僅適用於 patch 版本升級，major 版本升級需要額外的 migration 工具。**
- **兩個 Container 不能同時存取同一份資料——務必先停止舊 Container。**

---

## 🔑 關鍵字

Named Volume, PostgreSQL, Docker Hub, Migration, Patch Version
