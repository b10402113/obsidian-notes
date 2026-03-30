# E-Commerce Architecture

## 定義

E-Commerce Architecture 是課程中用來說明 Monolithic 與 Microservices 差異的案例應用：
- **Monolithic**：所有功能（users、products、orders）都在同一個 codebase，資料都在同一個資料庫
- **Microservices**：用戶註冊 → Service A + Database、 商品管理 → Service B + Database、訂單處理 → Service C + Database

## 核心問題

在 microservices 架構下，如果要新增 Service D 來查詢「某位使用者曾購買的所有商品」，Service D 不能直接讀取 Service A、B、C 的資料庫——這就是 [[Data Management]] 的核心挑戰。

## 相關概念

- [[Microservice]]
- [[Monolithic]]
- [[Database per Service]]
- [[Synchronous Communication]]

## 來源

- [[3_Data_in_Microservices]]
