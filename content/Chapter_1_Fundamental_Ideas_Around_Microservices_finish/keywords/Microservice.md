# Microservice

## 定義

Microservice（微服務）是一種將應用程式拆分為多個獨立服務的架構模式，每個 service 只負責實作一個功能（feature），並擁有自己完整的資源。

> **A single microservice contains all the routing, all the middleware, all the business logic, and database access required to implement ONE feature of our application.**

## 核心特性

- 每個 service 100% 獨立運作
- 擁有獨立的 database（[[Database per Service]]）
- 可獨立部署與擴展
- 故障隔離：單一 service 崩潰不會影響其他 service

## 相關概念

- [[Monolithic]]
- [[Feature Isolation]]
- [[Self-Contained Service]]
- [[Fault Tolerance]]

## 來源

- [[1_How_to_Get_Help]]
