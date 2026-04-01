---
title: "Chapter 20: DB 총집합 — PostgreSQL, Redis, MongoDB + 종합 비교"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, postgresql, redis, mongodb, database, comparison]
description: "NCP NCE 자격증 대비 학습 자료 - PostgreSQL, Redis, MongoDB와 주요 DB 서비스 비교"
---
> **학습 목표**: PostgreSQL/Redis/MongoDB의 특성과 스펙, 그리고 전체 DB 서비스의 포트/스토리지/Slave/백업 숫자를 완벽히 암기한다.

---

## 이야기: DB 종류가 이렇게 많아?

클라우드빵집이 성장하면서 다양한 데이터를 저장해야 했다.

민준이 고민했다.

> "회원 정보는 MySQL에 넣었는데, 실시간 세션은 어디에? 빵 리뷰 댓글은? 검색 로그는?"

재현이 화이트보드에 쭉 적어내려갔다.

> "용도에 따라 DB가 달라. 캐시는 Redis, 문서는 MongoDB, 관계형이 필요하면 PostgreSQL."

---

## 🔑 핵심 개념 1: Cloud DB for PostgreSQL

**PostgreSQL** = 오픈소스 관계형 DB의 명품. 복잡한 쿼리와 트랜잭션에 강함.

### 기본 스펙

| 항목 | 내용 |
|------|------|
| **DB 엔진** | PostgreSQL |
| **최대 스토리지** | **6TB** (자동 확장) |
| **자동 백업 보관** | 최대 **30일** |
| **Slave DB** | 최대 **5대** |
| **기본 포트** | **5432** |

### 특징

```
[PostgreSQL Slave 구조]

Primary DB (쓰기 + 읽기)
    │
    ├─ Slave 1 (읽기 전용)
    ├─ Slave 2 (읽기 전용)
    │         ⋮
    └─ Slave 5 (읽기 전용) ← 최대 5대
```

- MySQL의 Read Replica와 유사하게 **실시간 동기화** 지원
- HA 구성으로 자동 Fail-over 지원
- 시점 복원 (Point-In-Time Recovery) 지원

---

## 🔑 핵심 개념 2: Cloud DB for Redis (⭐⭐ 빈출!)

**Redis** = **Re**mote **Di**ctionary **S**erver. 메모리 기반 Key-Value 저장소.

> 💡 **사용 목적**: 빠른 읽기/쓰기가 필요한 **캐시, 세션, 실시간 순위 데이터**

### 기본 스펙

| 항목 | 내용 |
|------|------|
| **데이터 구조** | **Key-Value** (메모리 In-Memory) |
| **기본 포트** | **6379** |
| **자동 백업 보관** | **7일** ← ⚠️ 다른 DB는 30일! Redis만 7일! |
| **클러스터 구성** | Cluster mode (샤딩) |

### Redis Cluster 구성 (⭐⭐ 최빈출!)

```
[Redis Cluster 구조]

Shard 1 (Primary)  Shard 2 (Primary)  Shard 3 (Primary)
    └─ Replica 1       └─ Replica 1       └─ Replica 1
    └─ Replica 2       └─ Replica 2       └─ Replica 2
    └─ Replica 3       └─ Replica 3       └─ Replica 3
    └─ Replica 4       └─ Replica 4       └─ Replica 4

← Shard 당 최대 4개 Replica ←
← 전체 Shard 수: 최소 3 ~ 최대 10 ←
```

| 항목 | 내용 |
|------|------|
| **Shard 최소 ~ 최대** | **3 ~ 10개** |
| **Shard당 Replica 최대** | **4개** |

> ⚠️ **시험 함정**: Redis 백업은 **7일** (나머지 DB는 모두 30일!)

### Redis 특성

```
[메모리 기반 DB 특성]

장점:
- 초고속 읽기/쓰기 (마이크로초 단위)
- 세션 관리, 실시간 캐시에 최적

단점:
- 메모리 용량 한계
- 영속성(Persistence)이 상대적으로 약함
- 비용이 일반 DB보다 높음
```

---

## 🔑 핵심 개념 3: Cloud DB for MongoDB

**MongoDB** = 문서(Document) 기반 NoSQL DB. JSON 형태로 데이터 저장.

### 기본 스펙

| 항목 | 내용 |
|------|------|
| **DB 타입** | **Document DB** (JSON/BSON) |
| **기본 포트** | **17017** ← ⚠️ 일반 MongoDB 27017과 다름! |
| **최대 스토리지** | **2TB** |
| **Slave DB** | 최대 **7대** ← 가장 많음! |
| **자동 백업 보관** | 최대 **30일** |

> ⚠️ **시험 함정**: NCP MongoDB 포트는 **17017** (일반 MongoDB 27017과 다름!)

### MongoDB 특성

```
[Document DB 구조 예시]

{
    "_id": "001",
    "name": "클라우드빵집",
    "products": [
        {"name": "크로아상", "price": 3500},
        {"name": "식빵", "price": 2000}
    ],
    "address": {
        "city": "서울",
        "district": "강남"
    }
}
```

- 스키마리스(Schema-less): 컬럼 추가 없이 유연한 데이터 저장
- 중첩 구조(Nested) 데이터에 강함
- 빠른 개발 속도 → 스타트업, 콘텐츠 서비스에 적합

---

## 🔑 핵심 개념 4: 전체 DB 종합 비교표 (⭐⭐⭐ 최빈출!)

| DB | 포트 | 최대 스토리지 | 최대 Slave | 백업 보관 | 특징 |
|----|------|-------------|-----------|---------|------|
| **MySQL** | **3306** | **6TB** | **10대** | 30일 | Read Replica, 실시간 동기화 |
| **MSSQL** | **1433** | **2TB** | **5대** | 30일 | Log Shipping, BI/Batch 전용 |
| **PostgreSQL** | **5432** | **6TB** | **5대** | 30일 | 복잡한 쿼리, 오픈소스 |
| **Redis** | **6379** | - (메모리) | 샤드당 **4대** | **7일** ⚠️ | Key-Value, 캐시/세션 |
| **MongoDB** | **17017** | **2TB** | **7대** | 30일 | Document DB, JSON |

### 포트번호 암기법

```
MySQL  = 3306  → "쓰리(3) 쓰리(3) 공(0) 육(6)"
MSSQL  = 1433  → "하나(1) 사(4) 쓰리(3) 쓰리(3)"
Pgsql  = 5432  → "오(5) 사(4) 쓰리(3) 이(2)"
Redis  = 6379  → "육(6) 쓰리(3) 칠(7) 구(9)"
Mongo  = 17017 → "일(1) 칠(7) 공(0) 일(1) 칠(7)" ← NCP 전용!
```

### Slave 수 암기법

```
MySQL  10대 > MongoDB 7대 > MSSQL 5대 = PostgreSQL 5대 > Redis (샤드당 4대)
```

---

## 🔑 핵심 개념 5: DB 유형별 용도 선택 가이드

```
[어떤 DB를 선택해야 할까?]

관계형 데이터 (JOIN, 트랜잭션)
    ├─ Windows/.NET 기반 → MSSQL
    ├─ 일반 웹 서비스 → MySQL
    └─ 복잡한 쿼리, 오픈소스 선호 → PostgreSQL

비관계형 데이터 (NoSQL)
    ├─ 실시간 캐시, 세션 → Redis
    └─ 유연한 문서 구조, JSON → MongoDB

읽기 부하 분산
    ├─ 실시간 서비스 트래픽 → MySQL Read Replica (최대 10대)
    └─ BI/Batch 분석 → MSSQL Slave (Log Shipping)
```

---

## 📝 시험 대비 체크리스트

**PostgreSQL:**
- [ ] 포트: **5432**
- [ ] 최대 스토리지: **6TB**
- [ ] Slave 최대: **5대**
- [ ] 백업 보관: **30일**

**Redis:**
- [ ] 포트: **6379**
- [ ] 타입: **Key-Value (In-Memory)**
- [ ] Cluster: Shard **3~10개** / Shard당 Replica **최대 4개**
- [ ] 백업 보관: **7일** (⚠️ 다른 DB와 다름!)

**MongoDB:**
- [ ] 포트: **17017** (⚠️ 일반 27017과 다름!)
- [ ] 타입: **Document DB**
- [ ] 최대 스토리지: **2TB**
- [ ] Slave 최대: **7대** (전체 DB 중 가장 많음)
- [ ] 백업 보관: **30일**

**DB 종합:**
- [ ] Slave 순서: MySQL(10) > MongoDB(7) > MSSQL=PG(5) > Redis(샤드당 4)
- [ ] Storage 순서: MySQL=PG(6TB) > MSSQL=Mongo(2TB)
- [ ] 백업 7일인 것: **Redis만!** 나머지는 모두 30일

---

## 🧠 암기 핵심 문장

> "**MySQL 10대, MongoDB 7대, MSSQL·PG 5대, Redis 샤드당 4대**"  
> "**Redis만 7일 백업, 나머지는 30일**"  
> "**MongoDB NCP 포트 = 17017 (일반 27017 아님!)**"

---

*다음 챕터: `21_BigData_CloudSearch_Hadoop.md` → 빅데이터 분석 플랫폼*
