---
title: "Chapter 19: 데이터의 집 — Cloud DB for MySQL & MSSQL"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, mysql, mssql, cloud-db, replica, log-shipping]
description: "NCP NCE 자격증 대비 학습 자료 - Cloud DB for MySQL과 MSSQL 기능 및 제약 비교"
---
> **학습 목표**: MySQL과 MSSQL의 스펙, Fail-over 동작, Read Replica vs Log Shipping 차이를 완벽히 구분한다.

---

## 이야기: 직접 DB를 관리하다가 지친 민준

클라우드빵집 DB를 직접 서버에 설치해 운영했던 민준.
밤마다 백업하고, 버전 업그레이드하고, 장애 시 복구하다가 지쳐버렸다.

> "DB 관리가 본업보다 더 힘들어. 이거 자동으로 해주는 서비스 없어?"

재현이 웃으며 말했다.

> "**Cloud DB** 써. 백업, Fail-over, Read Replica 전부 자동이야."

---

## 🔑 핵심 개념 1: Cloud DB for MySQL

### 기본 스펙

| 항목 | 내용 |
|------|------|
| **DB 엔진** | MySQL **8.0.x** |
| **서버 타입** | High CPU, Standard, High Memory (VPC 환경) |
| **스토리지** | SSD 기반, 기본 **10GB** 시작 |
| **최대 스토리지** | 자동 확장 → 최대 **6,000GB (6TB)** |
| **최대 스펙** | **32 vCPU, 256GB RAM** |
| **구성 방식** | **HA (고가용성)** 또는 **Stand-alone** |

> ⚠️ Stand-alone으로 변경 시 → 시점 복원, 이중화 등 HA 특화 기능 **불가**

---

### 핵심 기능 4가지 (⭐ 시험 단골!)

#### 1. 자동 백업
- 지정된 시간에 **자동 백업** 수행
- 시점 복원 (Point-In-Time Recovery) 지원

#### 2. 자동 Fail-over (⭐⭐)

```
[MySQL Fail-over 동작]

별도 모니터 서버
    ↓ Master DB 상태 체크 (1분 단위)
    
Master DB 응답 없음 감지
    ↓
Standby Master DB로 자동 절체 (Fail-over)

→ 1분 단위 모니터링!
```

> ⚠️ **시험 포인트**: MySQL Fail-over는 **별도 모니터 서버**가 **1분 단위**로 체크!

#### 3. Read Replica (읽기 복제본)

```
[Read Replica 구조]

Master DB (쓰기 + 읽기)
    │
    ├─ Slave DB 1 (읽기 전용)
    ├─ Slave DB 2 (읽기 전용)
    │         ⋮
    └─ Slave DB 10 (읽기 전용) ← 최대 10대
```

- 읽기 부하 분산을 위한 Slave DB
- 최대 **10대**까지 확장 가능
- 실시간 동기화 → 즉시 서비스 가능

#### 4. DB 엔진 업그레이드

```
업그레이드 순서:
Recovery → Slave → Master (1대씩, 대당 약 1분 내외)

방식: Master → Standby Master로 먼저 전환 후 업그레이드
     → 서비스 중단 최소화
```

---

### 주요 Config 파라미터 (MySQL)

| 파라미터 | 기본값 | 설명 |
|---------|--------|------|
| `innodb_buffer_pool_size` | 1073741824 (Byte) | 캐시 역할. 메모리의 50~80%. **재시작 후 적용** |
| `max_connections` | **3000** | 최대 동시 접속 수. 초과 시 'Too many connections' |
| `wait_timeout` | **28800초 (8시간)** | Connection 유지 시간. 많을 때 줄임 |
| `Connect_timeout` | **10초** | 초기 연결 대기 시간 |
| `general_log` | - | 전체 쿼리 로그 (테이블 형태) |
| `long_query_time` | **1초** | Slow Query 기준 시간 |
| `Autocommit` | **On** | 자동 커밋 여부 |
| `character_set_server` | **utf8mb4** | DB 문자셋 (4바이트, 이모지 지원) |

---

## 🔑 핵심 개념 2: Cloud DB for MSSQL

### 기본 스펙

| 항목 | 내용 |
|------|------|
| **DB 엔진** | Microsoft SQL Server |
| **최대 스펙** | **24 vCPU, 128GB RAM** |
| **최대 스토리지** | **2TB** |
| **자동 백업 보관** | 최대 **30일** |
| **Edition** | Standard Edition도 자동 Fail-over 지원 |

---

### MySQL vs MSSQL의 가장 큰 차이: Slave DB 동작 방식 (⭐⭐ 최빈출 함정!)

| 항목 | MySQL Read Replica | MSSQL Slave DB |
|------|------------------|----------------|
| **동기화 방식** | 실시간 복제 | **Log Shipping (로그 배송)** |
| **실시간 읽기** | ✅ 가능 | ❌ **로그 복원 중 읽기 불가** |
| **서비스 용도** | 실시간 트래픽 분산 | **BI/Batch 작업 전용** |
| **최대 대수** | **10대** | **5대** |
| **읽기 가능 시간** | 24시간 | 하루 최대 **20시간** 설정 |

> ⚠️ **시험 함정**: "MSSQL Slave DB는 읽기 분산에 쓸 수 있다?" → **X**  
> MSSQL Slave는 **Log Shipping** 방식. 트랜잭션 로그 복원 중 읽기 불가.  
> → **BI (Business Intelligence) 분석, Batch 작업 용도**로만 사용!

```
[MSSQL Log Shipping 동작]
Master DB → 트랜잭션 로그 백업 생성
         → Slave DB에 로그 전송
         → Slave에서 로그 Restore(복원)
         → 복원 중에는 데이터 읽기 불가!
```

### MSSQL 주요 Config

| 파라미터 | 기본값 | 설명 |
|---------|--------|------|
| `user connection` | **32767** | 최대 동시 접속 수 |
| `remote access` | **0 (허용)** | 원격 저장 프로시저 실행 권한 |
| `remote query timeout` | **600초 (10분)** | Connection 유지 시간 |

### MSSQL Slave DB 특이사항

- Slave 스펙 변경 시 **Principal + Mirror 서버도 함께** 스펙 변경
- 읽기 가능 Slave: 최대 **5대**
- 읽기 가능 시간: 하루 최대 **20시간**

### MSSQL 쿼리 분석 (Query Analysis)

```
[버블 차트 형태 제공 - 1분 단위]
X축: 일별 쿼리 수행 횟수 합계
Y축: 일별 CPU 소모량 합계
버블 크기: 메모리 읽기 수
```

---

## 📝 시험 대비 체크리스트

**MySQL:**
- [ ] 최대 스토리지: **6TB** (자동 확장)
- [ ] 최대 스펙: **32 vCPU, 256GB**
- [ ] Fail-over: 별도 모니터 서버, **1분 단위** 체크
- [ ] Read Replica: 최대 **10대**
- [ ] 업그레이드 순서: **Recovery → Slave → Master**
- [ ] `max_connections` 기본값: **3000**
- [ ] `wait_timeout` 기본값: **28800초**
- [ ] `character_set_server` 기본값: **utf8mb4**

**MSSQL:**
- [ ] 최대 스토리지: **2TB**
- [ ] 최대 스펙: **24 vCPU, 128GB**
- [ ] Slave 방식: **Log Shipping** (실시간 읽기 **불가**)
- [ ] MSSQL Slave 용도: **BI/Batch 전용**
- [ ] 읽기 가능 Slave: 최대 **5대**
- [ ] 읽기 가능 시간: 하루 최대 **20시간**
- [ ] `user connection` 기본값: **32767**
- [ ] `remote query timeout`: **600초**

---

## 🧠 암기 핵심 문장

> "**MySQL Read Replica = 실시간 10대 / MSSQL Slave = Log Shipping, BI/Batch, 5대**"  
> "**MySQL Fail-over = 1분 단위 모니터링 / MSSQL = Standard도 자동 Fail-over**"

---

*다음 챕터: `20_데이터베이스_비교.md` → PostgreSQL, Redis, MongoDB + DB 종합 비교*
