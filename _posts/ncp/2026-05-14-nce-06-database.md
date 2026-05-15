---
title: "[NCE] 06. Cloud Database"
date: 2026-05-14 09:50:00 +0900
categories: [NCP, NCE]
tags: [NCP, NCE, MySQL, MSSQL, MongoDB, PostgreSQL, Redis, Cloud DB, Failover]
description: "Cloud DB for MySQL/MSSQL/MongoDB/PostgreSQL/Redis 스펙 비교, Failover, 백업 정책, Config 주요 파라미터 정리"
---

## 핵심 요약

- MySQL: 32vCPU/256GB/6TB, Slave 최대 10대, Failover ~1분, 백업 최대 30일, 기본 01:00, 포트 3306
- MSSQL: 24vCPU/128GB/2TB, Log Shipping 방식, 읽기 최대 20시간, 최대 5대, 포트 1433
- MongoDB: ReplicaSet 최대 7대, 포트 17017
- PostgreSQL: Read Replica 최대 5대, 포트 5432
- Redis(Cache): 포트 6379, 백업 **최대 7일** (다른 CDB는 30일), Cluster 3~10 shard

## DB Replication 필요성

웹 서버는 LB + Auto Scaling으로 수평 확장 가능하지만, **DB가 단일 노드면 SPOF**가 된다.

- **DB Replication**: Master(쓰기) / Slave(읽기) 분리로 SPOF 해결 + 읽기 부하 분산
- Slave 추가로 읽기 트래픽 대응

## Cloud DB for MySQL

| 항목 | 값 |
|------|-----|
| 최대 vCPU | **32개** |
| 최대 Memory | **256 GB** |
| 최대 Storage | **6 TB** |
| Read Replica (Slave) | **최대 10대** |
| 기본 포트 | **3306** |
| Failover 시간 | **약 1분** |
| 백업 보관 | **최대 30일** (최소 1일) |
| 백업 기본 시간 | **01:00** |

### HA 구성

- **Master DB** (읽기/쓰기) + **Standby Master** (다른 Zone, 장애 시 자동 승격) + **Read Replica** (읽기 전용)
- Failover: 모니터 서버가 1분간 쿼리 응답 없으면 Standby Master로 자동 전환

### 백업 및 복원

- 시점 복원(Point-in-Time Recovery): **분 단위**까지 지원
- 복원 시 신규 VM 생성, Recovery 모드로 부팅 (데이터 조회만 가능)

### 주요 Config

| 항목 | 기본값 | 설명 |
|------|--------|------|
| `innodb_buffer_pool_size` | ~1 GB | 메모리의 **50~80%** 권장, DB 재시작 필요 |
| `max_connections` | **3000** | 초과 시 Too many connections 오류 |
| `wait_timeout` | **28800초 (8시간)** | Connection 연결 유지 시간 |
| `connect_timeout` | **10초** | 초기 연결 대기 시간 |
| `long_query_time` | **1초** | Slow Query 기준 시간 |
| `character_set_server` | **utf8mb4** | 4바이트 문자셋 |

## Cloud DB for MSSQL

| 항목 | 값 |
|------|-----|
| 최대 vCPU | **24개** |
| 최대 Memory | **128 GB** |
| 최대 Storage | **2 TB** |
| 기본 포트 | **1433** |
| 백업 보관 | **최대 30일** |
| Slave 방식 | **Log Shipping** |
| 읽기 가능 시간 | **최대 20시간** 설정 가능 |
| Slave 최대 | **5대** |

- Log Shipping: transaction log 백업을 restore하는 동안 읽기 **불가**
- **일반 서비스 불가**, 매일 주기적인 **BI/Batch 작업**에 적합

## Cloud DB for MongoDB

| 항목 | 값 |
|------|-----|
| 기본 포트 | **17017** |
| Replica Set 최대 | **7대** |
| 기본 스토리지 | 50 GB |
| 최대 스토리지 | 2 TB |
| 백업 보관 | **최대 30일** |

- Sharding 또는 ReplicaSet 방식 선택 가능

## Cloud DB for PostgreSQL

| 항목 | 값 |
|------|-----|
| 기본 포트 | **5432** |
| Read Replica 최대 | **5대** |
| 최대 스토리지 | 6 TB |
| 백업 보관 | **최대 30일** |

## Cloud DB for Cache (Redis)

| 항목 | 값 |
|------|-----|
| 기본 포트 | **6379** |
| 백업 보관 | **최대 7일** ← 다른 CDB(30일)와 차이! |
| Cluster 샤드 | **최소 3개 ~ 최대 10개** |
| 샤드당 Slave | 최대 4대 |

### 구성 형태

| 구성 | 설명 |
|------|------|
| **Simple** | 단일 Master |
| **Simple + HA** | Master + Standby |
| **Cluster** | 샤드 3~10개 |
| **Cluster + HA** | Cluster + Standby |

- Auto Sharding 지원
- VPC 내 **Private Subnet**에 구성

## Cloud DB 제품 비교

| 구분 | MySQL | MSSQL | PostgreSQL | MongoDB | Redis |
|------|-------|-------|------------|---------|-------|
| 포트 | **3306** | **1433** | **5432** | **17017** | **6379** |
| 최대 vCPU | 32 | 24 | - | - | - |
| 최대 Storage | 6TB | 2TB | 6TB | 2TB | - |
| Replica 최대 | **10대** | **5대** | **5대** | **7대** | 샤드당 4대 |
| 백업 보관 | **30일** | **30일** | **30일** | **30일** | **7일** |

## 시험 암기 포인트

- **MySQL Failover**: 약 1분 (1분간 쿼리 응답 없을 때)
- **MySQL**: 32 vCPU / 256GB / 6TB / Slave 10대 / 기본 백업 01:00
- **MSSQL**: 24 vCPU / 128GB / 2TB / Log Shipping / 읽기 최대 20시간 / 5대
- **MongoDB**: 포트 17017, ReplicaSet 최대 7대
- **PostgreSQL**: 포트 5432, Read Replica 5대
- **Redis**: 포트 6379, 백업 **7일** (다른 DB는 30일), Cluster 3~10 shard
- **innodb_buffer_pool_size**: 메모리 50~80% 권장
- **max_connections**: 기본 3000
- **wait_timeout**: 기본 28800초 (8시간)

