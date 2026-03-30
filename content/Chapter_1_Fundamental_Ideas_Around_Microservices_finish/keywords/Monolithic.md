# Monolithic

## 定義

Monolithic（單體式架構）是一種傳統的軟體架構模式，將應用程式的所有功能——包括 routing、middleware、business logic、資料庫存取——全部集中在同一個 codebase 裡，部署時以單一離散單元（single unit）進行。

> **A monolith contains all the routing, all the middleware, all the business logic, and all the database access code required to implement ALL features of our application.**

## 核心特性

- 單點故障（Single Point of Failure）的典型代表
- 所有功能共享同一個 codebase
- 擴展時需要複製整個 application

## 相關概念

- [[Microservice]]
- [[Single Unit Deployment]]
- [[Monolithic vs. Microservices]]

## 來源

- [[1_How_to_Get_Help]]
