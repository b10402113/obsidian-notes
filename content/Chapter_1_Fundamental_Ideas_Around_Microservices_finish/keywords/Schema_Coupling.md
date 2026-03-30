# Schema Coupling

## 定義

Schema Coupling（Schema 耦合）指的是當一個 service 的程式碼依賴另一個 service 的資料庫結構時所產生的問題。如果後者修改了 schema（如將 `name` 改名為 `first_name`），前者若不知情就會產生難以追蹤的 bug。

## 解決方案

[[Database per Service]] 模式透過嚴禁跨 service 直接讀取資料庫，來消除 schema coupling。

## 相關概念

- [[Database per Service]]
- [[Service Independence]]

## 來源

- [[3_Data_in_Microservices]]
