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

> "PostgreSQL 데이터베이스를 쉽고 간편하게 구축하고 관리." — NCP 공식 설명

### 기본 스펙

| 항목 | 내용 |
|------|------|
| **DB 엔진** | PostgreSQL |
| **초기 스토리지** | **10GB**부터 시작 |
| **최대 스토리지** | **6TB** (10GB씩 자동 확장) |
| **자동 백업 보관** | 최대 **30일** |
| **Slave DB** | 최대 **5대** (Read Replica) |
| **기본 포트** | **5432** |
| **서버 타입** | Standard / High Memory / High CPU |

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

- 네이버 다양한 서비스에서 오랜 시간 **검증된 최적화 설정**을 기본 제공
- MySQL의 Read Replica와 유사하게 **실시간 동기화** 지원 (최대 5대까지 Read Replica 확장, Load Balancer 연동으로 읽기 부하 분산 가능)
- HA 구성으로 **자동 Fail-over** 지원
- 매일 자동으로 DB 백업 진행, 데이터 최대 30일 보관
- 설치 후 즉시 DB 모니터링 이용 가능, PostgreSQL 로그를 콘솔 웹 화면에서 확인

---

## 🔑 핵심 개념 2: Cloud DB for Redis (⭐⭐ 빈출!)

**Redis** = **Re**mote **Di**ctionary **S**erver. 메모리 기반 Key-Value 저장소.

> 💡 **사용 목적**: 빠른 읽기/쓰기가 필요한 **캐시, 세션, 실시간 순위 데이터**

NCP에서는 Cloud DB for Redis를 **"Cloud DB for Cache"**라는 이름으로도 제공한다. Redis 캐시를 쉽고 간편하게 구축하고 자동으로 관리하는 서비스다.

### 기본 스펙

| 항목 | 내용 |
|------|------|
| **데이터 구조** | **Key-Value** (메모리 In-Memory) |
| **기본 포트** | **6379** |
| **자동 백업 보관** | **7일** ← ⚠️ 다른 DB는 30일! Redis만 7일! |
| **클러스터 구성** | Redis Simple / Redis Cluster 모두 제공 |
| **서버 타입** | Standard / High Memory |

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
| **네트워크 구성** | VPC 내 Private Subnet에 구성 |
| **HA 구성** | 지원 |
| **Auto Sharding** | 지원 |

> ⚠️ **시험 함정**: Redis 백업은 **7일** (나머지 DB는 모두 30일!)

### Redis 특성

```
[메모리 기반 DB 특성]

장점:
- 초고속 읽기/쓰기 (마이크로초 단위)
- 세션 관리, 실시간 캐시에 최적
- Redis가 미제공하는 자동 Fail-over를 NCP가 자체 개발하여 제공
  → 장애 발생 시에도 안정적인 서비스 가능

단점:
- 메모리 용량 한계
- 영속성(Persistence)이 상대적으로 약함
- 비용이 일반 DB보다 높음
```

### Redis 운영 상세: Config 관리

Redis 운영에서 중요한 것 중 하나가 **Config Group**을 통한 설정 관리다.

CloudDB for Redis에서 지원하는 설정 변수 목록 (주요 항목):

| 설정 변수 | 설명 |
|-----------|------|
| `tcp-backlog` | TCP 연결 백로그 크기 |
| `timeout` | 클라이언트 연결 타임아웃 |
| `tcp-keepalive` | TCP keepalive 설정 |
| `rdbcompression` | RDB 압축 여부 |
| `rdbchecksum` | RDB 체크섬 여부 |
| `slave-read-only` | Slave 읽기 전용 여부 |
| `maxclients` | 최대 클라이언트 수 |
| `maxmemory-policy` | 메모리 초과 시 데이터 처리 정책 |
| `cluster-require-full-coverage` | 클러스터 전체 커버리지 요구 여부 |
| `slowlog-log-slower-than` | 슬로우 로그 기준 시간 |
| `slowlog-max-len` | 슬로우 로그 최대 길이 |
| `hash-max-ziplist-entries` | Hash 자료구조 ziplist 최대 항목 수 |
| `zset-max-ziplist-entries` | Sorted Set ziplist 최대 항목 수 |

### Redis 운영 상세: 백업 설정 (⭐ 빈출!)

Redis 백업 설정에는 세 가지 핵심 사항이 있다:

1. **백업 파일 저장 위치**: 별도의 **데이터 스토리지**에 저장됨 (운영 서버와 분리)
2. **백업 시간 설정**: 원하는 시간에 백업이 진행되도록 설정 가능
3. **자동 백업 옵션**: "자동"으로 선택할 경우 **임의의 시간**에 백업 진행

> 💡 **복구 시 주의**: 백업 파일은 백업 완료 시점의 데이터를 보관하므로, 복구 데이터베이스는 **백업 완료 시점 기준의 데이터**로 복구됨

### Redis 운영 상세: 모니터링

Redis 서버에 대한 두 가지 Dashboard를 제공하며, **최근 4주 이내**의 모니터링 지표를 확인할 수 있다.

**Redis Dashboard 주요 지표:**

| 그래프명 | 단위 | 설명 |
|---------|------|------|
| MemoryFragmentationRatio | ratio (%) | OS 사용 메모리와 Redis 할당 메모리의 비율 |
| InstantaneousOPSPerSec | avg activity/min | Redis Server에서 처리한 총 명령 횟수 |
| EvictedKeys | avg activity/min | maxmemory-policy에 의해 발생한 evicted key 수 |
| RedisUsedMemory | kb/min | 사용하고 있는 메모리의 양 |
| KeyspaceHits | avg activity/min | 지정한 보고 간격 동안 성공한 키 조회 수 |
| KeySpaceMisses | avg activity/min | 지정한 보고 간격 동안 실패한 키 조회 수 |
| ExpiredKeys | avg activity/min | expire된 key 수 |

**OS Dashboard 주요 지표:**

| 그래프명 | 단위 | 설명 |
|---------|------|------|
| CPUUsage | used (%) | CPU 사용량 |
| Swap | kb/min | Swap 메모리 발생량 |
| Network | kb/min | NIC의 초당 in/outbound byte |
| LoadAverage | - | Load Average 상태값 |

---

## 🔑 핵심 개념 3: Cloud DB for MongoDB

**MongoDB** = 문서(Document) 기반 NoSQL DB. JSON 형태로 데이터 저장.

NoSQL의 대표 격인 MongoDB를 쉽고 간편하게 사용할 수 있는 **완전 관리형 서비스**다.

### 기본 스펙

| 항목 | 내용 |
|------|------|
| **DB 타입** | **Document DB** (JSON/BSON) |
| **기본 포트** | **17017** ← ⚠️ 일반 MongoDB 27017과 다름! |
| **초기 스토리지** | **50GB**부터 시작 |
| **최대 스토리지** | **2TB** (10GB씩 자동 확장) |
| **ReplicaSet 최대** | 최대 **7대** ← 전체 DB 중 가장 많음! |
| **자동 백업 보관** | 최대 **30일** |
| **서버 타입** | Standard / High CPU |

> ⚠️ **시험 함정**: NCP MongoDB 포트는 **17017** (일반 MongoDB 27017과 다름!)

> ⚠️ **시험 함정**: MongoDB 초기 스토리지는 **50GB** (MySQL/PostgreSQL은 10GB, MSSQL은 100GB)

### MongoDB 구성 방식

MongoDB는 설치 과정에서 두 가지 구성 방식을 선택할 수 있다:

```
[MongoDB 구성 방식 선택]

방식 1: Sharded Cluster
    ├─ 대용량 데이터의 수평 분산
    └─ 여러 Shard에 데이터를 나누어 저장

방식 2: ReplicaSet
    ├─ 최대 7대까지 확장 가능
    └─ 읽기 부하 분산 기능 제공
```

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
- 자동 Fail-over 기능 제공 → Primary DB 장애 발생 시에도 안정적인 서비스 가능
- 빠른 개발 속도 → 스타트업, 콘텐츠 서비스에 적합
- 설치 후 즉시 DB 모니터링 이용 가능, MongoDB 로그를 콘솔 웹 화면에서 확인
- 장애 또는 이벤트 발생 시 사용자의 메일·SMS로 장애 알람

---

## 🔑 핵심 개념 4: Cloud DB for MSSQL 운영 보충

앞 챕터에서 MSSQL 기본 스펙을 다뤘다면, 여기서는 **운영 세부 사항**을 보충한다.

### Config Group 주요 설정

| 설정 항목 | 기본값 | 설명 |
|-----------|--------|------|
| `userconnection` (최대 동시접속 수) | **32767** | False 기본값으로 32767 |
| `remoteaccess` | **0** | 원격 서버에서 저장 프로시저 실행 제어 (0 = 원격 실행 허용) |
| `remotequerytimeout` | **600초 (10분)** | Connection 연결 유지 시간 |

### Slave DB 운영 특이사항

MSSQL Slave는 일반 DB와 다르게 동작한다. 이 차이가 시험에 나온다.

```
[MSSQL Slave = Log Shipping 방식]

1. transaction log 백업을 restore하는 동안 → 읽기 불가
2. 일반 서비스 트래픽에는 사용 불가
3. 매일 주기적인 BI 및 Batch 작업에 적합
4. 읽기 가능 시간: 시간 단위로 최대 20시간까지 설정 가능
5. 읽기 가능 시간대: 최대 20개 시간대 선택 가능
6. 읽기 가능 Slave: 최대 5대까지 생성 가능
```

> ⚠️ **시험 함정**: MSSQL Slave의 spec 변경 시, **Principal과 Mirror 서버도 함께 변경**됨

### 이벤트 설정

**Cloud Insight**를 통해 알람 항목과 임계치를 설정하여, 임계치를 넘은 이벤트에 대해 **메일과 SMS로 실시간 통보** 알람을 받을 수 있다.

### 쿼리 분석

- 쿼리 수행 횟수 대비 CPU 소모량과 메모리 읽기 수를 **버블 차트**로 표현
- X축: 해당 쿼리의 일별 수행 횟수 합계
- Y축: 해당 쿼리의 일별 CPU 소모량 합계
- 버블의 크기: 해당 쿼리의 메모리 읽기 수

---

## 🔑 핵심 개념 5: 전체 DB 종합 비교표 (⭐⭐⭐ 최빈출!)

| DB | 포트 | 초기 스토리지 | 최대 스토리지 | 최대 Slave | 백업 보관 | 특징 |
|----|------|------------|-------------|-----------|---------|------|
| **MySQL** | **3306** | **10GB** | **6TB** | **10대** | 30일 | Read Replica, 실시간 동기화 |
| **MSSQL** | **1433** | **100GB** | **2TB** | **5대** | 30일 | Log Shipping, BI/Batch 전용 |
| **PostgreSQL** | **5432** | **10GB** | **6TB** | **5대** | 30일 | 복잡한 쿼리, 오픈소스 |
| **Redis** | **6379** | - (메모리) | - (메모리) | 샤드당 **4대** | **7일** ⚠️ | Key-Value, 캐시/세션 |
| **MongoDB** | **17017** | **50GB** | **2TB** | **7대** | 30일 | Document DB, JSON |

> ⚠️ **초기 스토리지 차이**: MySQL·PostgreSQL(10GB) < MongoDB(50GB) < MSSQL(100GB)

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

### 서버 타입 정리

| DB | 서버 타입 |
|----|-----------|
| MySQL | Standard / High Memory / High CPU |
| MSSQL | Standard / High Memory / High CPU |
| PostgreSQL | Standard / High Memory / High CPU |
| Redis (Cache) | Standard / High Memory |
| MongoDB | Standard / High CPU |

---

## 🔑 핵심 개념 6: DB 유형별 용도 선택 가이드

```
[어떤 DB를 선택해야 할까?]

관계형 데이터 (JOIN, 트랜잭션)
    ├─ Windows/.NET 기반 → MSSQL
    ├─ 일반 웹 서비스 → MySQL
    └─ 복잡한 쿼리, 오픈소스 선호 → PostgreSQL

비관계형 데이터 (NoSQL)
    ├─ 실시간 캐시, 세션 → Redis (Cloud DB for Cache)
    └─ 유연한 문서 구조, JSON → MongoDB

읽기 부하 분산
    ├─ 실시간 서비스 트래픽 → MySQL Read Replica (최대 10대)
    ├─ 읽기 분산 + 고가용성 → PostgreSQL Read Replica (최대 5대, LB 연동)
    └─ BI/Batch 분석 (주기적) → MSSQL Slave (Log Shipping, 최대 20시간 읽기)

대용량 데이터 분산
    └─ MongoDB Sharded Cluster 또는 Redis Cluster (Shard 3~10개)
```

---

## 📝 시험 대비 체크리스트

**PostgreSQL:**
- [ ] 포트: **5432**
- [ ] 초기 스토리지: **10GB**, 최대 스토리지: **6TB**
- [ ] Slave 최대: **5대** (Read Replica, LB 연동 가능)
- [ ] 백업 보관: **30일**
- [ ] 서버 타입: Standard / High Memory / High CPU

**Redis (Cloud DB for Cache):**
- [ ] 포트: **6379**
- [ ] 타입: **Key-Value (In-Memory)**
- [ ] Cluster: Shard **3~10개** / Shard당 Replica **최대 4개**
- [ ] 백업 보관: **7일** (⚠️ 다른 DB와 다름!)
- [ ] 백업 파일: **별도 데이터 스토리지**에 저장
- [ ] 백업 시간: **직접 지정** 또는 **자동(임의 시간)**
- [ ] 모니터링: Redis Dashboard + OS Dashboard, **최근 4주** 이내 지표 확인

**MongoDB:**
- [ ] 포트: **17017** (⚠️ 일반 27017과 다름!)
- [ ] 타입: **Document DB**
- [ ] 초기 스토리지: **50GB**, 최대 스토리지: **2TB**
- [ ] ReplicaSet 최대: **7대** (전체 DB 중 가장 많음)
- [ ] 백업 보관: **30일**
- [ ] 구성 방식: **Sharded Cluster** 또는 **ReplicaSet** 선택 가능

**MSSQL 운영:**
- [ ] Config: userconnection 기본값 **32767**, remotequerytimeout 기본값 **600초**
- [ ] Slave 방식: **Log Shipping** (restore 중 읽기 불가)
- [ ] Slave 읽기 가능 시간: 최대 **20시간** / 최대 **20개 시간대**
- [ ] Slave 최대: **5대**

**DB 종합:**
- [ ] Slave 순서: MySQL(10) > MongoDB(7) > MSSQL=PG(5) > Redis(샤드당 4)
- [ ] Storage 순서: MySQL=PG(6TB) > MSSQL=Mongo(2TB)
- [ ] 초기 스토리지: MySQL=PG(10GB) < MongoDB(50GB) < MSSQL(100GB)
- [ ] 백업 7일인 것: **Redis만!** 나머지는 모두 30일

---

## 🧠 암기 핵심 문장

> "**MySQL 10대, MongoDB 7대, MSSQL·PG 5대, Redis 샤드당 4대**"  
> "**Redis만 7일 백업, 나머지는 30일**"  
> "**MongoDB NCP 포트 = 17017 (일반 27017 아님!)**"  
> "**Redis 백업은 별도 데이터 스토리지, 시간 지정 또는 자동(임의 시간)**"  
> "**MSSQL Slave = Log Shipping, restore 중 읽기 불가, BI/Batch 전용**"  
> "**MongoDB 초기 50GB, MSSQL 초기 100GB, MySQL·PG 초기 10GB**"

---

*다음 챕터: `21_BigData_CloudSearch_Hadoop.md` → 빅데이터 분석 플랫폼*
