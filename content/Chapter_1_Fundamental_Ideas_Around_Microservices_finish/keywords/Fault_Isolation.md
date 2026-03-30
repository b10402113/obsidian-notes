# Fault Isolation

## 定義

Fault Isolation（故障隔離）是 microservices 架構相較於 Monolithic 的關鍵優勢：當某個 service 故障時，該故障不會傳播到其他 service，系統整體仍能繼續運作。

## 與 Monolithic 的對比

Monolithic 的單點故障（Single Point of Failure）意味著一個元件損壞會導致整個系統崩潰，而 fault isolation 讓每個 service 的故障被限制在局部範圍內。

## 相關概念

- [[Microservice]]
- [[Fault Tolerance]]
- [[Single Point of Failure]]
- [[Fault Propagation]]

## 來源

- [[3_Data_in_Microservices]]
