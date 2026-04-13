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

그리고 덧붙였다.

> "그런데 왜 DB를 복제해야 하는지 알아야 제대로 설계할 수 있어."

---

## DB Replication이 필요한 이유

민준이 처음에는 단순하게 생각했다. "서버가 느리면 Auto Scaling으로 웹 서버 늘리면 되지 않아?"

재현이 화이트보드에 그림을 그렸다.

```
[Auto Scaling만 적용했을 때의 문제]

ALB
 ├── Web1 (Insert/Update/Delete → Master DB)
 ├── Web2 (Select → Master DB)
 └── Web3 (Select → Master DB)

→ 웹 서버는 늘었는데 DB는 한 대!
→ DB 병목 현상으로 결국 장애 발생

[DB Replication 적용 후]

ALB
 ├── Web1 (쓰기 → Master DB)
 └── Web2 (읽기 → Slave DB) ← 읽기/쓰기 분리!

Master ─[Replication]─ Slave
```

핵심은 두 가지다.

1. **SPOF 해결**: DB를 복제하여 Single Point of Failure 제거
2. **쓰기/읽기 분리**: Write(Insert/Update/Delete)는 Master, Read(Select)는 Slave로 분산하여 부하 감소. 트래픽 증가 시 Slave를 추가로 복제해 부하 분산

---

## CDB를 활용한 고가용성 구성

실제 NCP에서 권장하는 구성을 보면 이렇다.

```
한국 리전 (KR)
┌────────────────────────────────────────────┐
│  KR-1 Zone             KR-2 Zone           │
│  lab-vpc (100.0.0/16)                      │
│                                            │
│  Public Subnet         Public Subnet       │
│       ALB                                  │
│  Auto Scaling                              │
│       Web1                                 │
│                                            │
│  Private Subnet        Private Subnet      │
│  Master DB ──읽기/쓰기──  Standby           │
│  Read Replica ──읽기 전용                   │
└────────────────────────────────────────────┘
```

웹 레이어는 Auto Scaling + ALB로 수평 확장하고, DB 레이어는 Master-Standby 고가용성 구성에 Read Replica로 읽기 부하를 분산한다.

---

## 🔑 핵심 개념 1: Cloud DB for MySQL

### 기본 스펙

| 항목 | 내용 |
|------|------|
| **DB 엔진 버전** | MySQL **8.0.34**, **8.0.36**, **8.0.40**, **8.0.42** |
| **서버 타입** | High CPU, Standard, High Memory (VPC 환경) |
| **스토리지** | SSD 기반, 기본 **10GB** 시작 |
| **최대 스토리지** | 자동 확장 → 최대 **6,000GB (6TB)** |
| **최대 스펙** | **32 vCPU, 256GB RAM** |
| **구성 방식** | **HA (고가용성)** 또는 **Stand-alone** |
| **외부 접근** | Public domain 부여로 외부 접근 가능 |
| **최대 Slave** | 최대 **10대** |

> ⚠️ Stand-alone으로 변경 시 → 시점 복원, 이중화 등 HA 특화 기능 **불가**

---

### 핵심 기능 7가지 (⭐ 시험 단골!)

#### 1. 사용자 환경에 맞는 구성
- 최대 32개 vCPU, 256GB까지 제공
- 데이터 스토리지 최대 6TB까지 자동 확장

#### 2. 편리한 구성과 사용
- 클릭 몇 번으로 쉽게 DB 생성 및 구성
- 네이버 서비스를 제공하며 쌓은 경험이 반영된 파라미터 셋 제공

#### 3. 자동화된 DB 백업
- Cloud DB for MySQL 생성 시 설정한 백업 시간에 맞춰 자동 백업 진행
- 시점 복원 (Point-In-Time Recovery) 지원

#### 4. 자동 Fail-over (⭐⭐)

```
[MySQL Fail-over 동작]

별도 모니터 서버
    ↓ Master DB 상태 체크 (1분 단위)
    
Master DB 응답 없음 감지
(약 1분간 연속해서 쿼리 응답 없을 때)
    ↓
Standby Master DB로 자동 절체 (Fail-over)

→ 1분 단위 모니터링!
```

> ⚠️ **시험 포인트**: MySQL Fail-over는 **별도 모니터 서버**가 **1분 단위**로 체크!  
> 서버 장애나 DB 프로세스 다운 등의 상황에서 자동으로 Standby Master로 전환

#### 5. Read Replica (읽기 복제본) 확장

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
- 추가된 Slave DB로 읽기 부하 분산 가능

#### 6. 모니터링과 알람
- 모니터링과 알람 기능을 제공하여 문제 발생 시 관리자가 빠르게 파악하고 대응 가능

#### 7. HA → Stand-alone 변경 시 주의
- Stand-alone으로 변경하면 시점 복원, 이중화 등 HA 전용 기능 사용 불가

---

## CDB Operation 실무 가이드

민준이 실제로 콘솔을 열고 각 기능을 하나씩 탐험했다.

### Operation 1: DB Process List 확인

> "현재 누가 DB에 접속해 있는지 확인하고 싶어."

DB 서버에 현재 접속한 세션 리스트를 확인할 수 있다. MySQL에서 사용하는 `SHOW PROCESSLIST;`와 동일하다.

| 항목 | 설명 |
|------|------|
| **Session ID** | 접속한 세션의 고유 번호 |
| **USER** | 접속한 세션이 사용한 유저명 |
| **HOST(IP)** | 접속한 세션의 호스트 IP 주소 |
| **DB** | 접속한 세션이 사용한 DB명 |
| **Command** | 접속한 세션이 수행한 명령어 |
| **Time** | 수행한 명령어의 수행 시간 |
| **State** | 접속한 세션의 상태를 나타내는 값 |

**Kill Session** 기능:
- 선택된 Session ID를 강제로 종료
- 강제로 종료한 세션은 복구 불가능
- 하나의 세션만 선택 가능

> ⚠️ DB 서버에 직접 접속하여 사용자 계정으로 Processlist를 조회하는 경우, 로그인한 계정(USER\_ID + HOST(IP))에서 수행하는 Processlist만 확인 가능. 전체 Processlist가 보이지 않는 이유는 DB 서버에 접속한 계정만 조회되기 때문이다.

---

### Operation 2: Slave DB Replication 확인

> "Slave DB가 제대로 동기화되고 있는지 어떻게 확인해?"

- Slave 서버의 경우 Replication 상태를 확인할 수 있다 (Master 서버는 표시되지 않음)
- MySQL에서 사용하는 `SHOW SLAVE STATUS;` 명령어와 동일한 결과를 제공

**제공 기능:**

| 기능 | 설명 |
|------|------|
| **Skip Replication Error** | Slave DB에서 Replication 오류가 발생한 Query를 건너뛰어 오류를 조치. 이 과정에서 Master DB와 데이터 불일치가 발생할 수 있음 |
| **Slave DB 재설치** | Slave DB를 다시 설치. DB 접속 도메인은 변경되지 않으며, 재설치 진행 중 DB 서버 접속 불가. 데이터 사이즈에 따라 수십 분~수 시간 소요 가능 |

> ⚠️ **Replication 지연 주의**: Master에서 Write가 오래 수행되는 쿼리가 있거나 과도하게 많은 Write가 발생하는 경우 Replication 지연이 발생할 수 있다.  
> Replication 지연에 대한 알람은 **Event 메뉴**에서 등록 가능 (알람 항목: `Replication`, 알람 세부 항목: `Delay`)

---

### Operation 3: DB 서버 로그 확인

DB 서버 상세보기 > **DB Server Logs** 탭에서 로그 파일을 확인할 수 있다.

**제공 로그 종류:**
- Binary Log
- Error Log
- Slow Log
- General Log
- Audit Log

모든 로그 파일은 **Object Storage로 전송** 가능하다.

**주요 에러 로그 패턴:**

| 로그 메시지 | 의미 및 조치 |
|------------|-------------|
| `Access denied for user` | DB 패스워드 오류 또는 DB 접근 허용이 되지 않은 계정. 접근 IP, DB User, 패스워드 확인 필요 |
| `Aborted connection` | `wait_timeout` 또는 `interactive_timeout` 시간 동안 idle 상태가 유지되어 DB Server에서 종료된 세션 로그. MySQL Client에서 `mysql_close()` 호출 없이 종료되는지 DB Connection 설정 점검 필요 |

---

### Operation 4: DB 백업 설정 및 복원 (⭐)

> "백업은 언제, 얼마나 보관돼?"

**백업 정책:**
- 백업은 하루에 **한 번** 매일 수행
- 보관 기간: 최소 **1일** ~ 최대 **30일** (사용자 설정)
- 백업 시간을 설정하지 않은 경우 기본값인 **01:00**에 수행
- 백업 최대 용량 제한은 없으며, 사용 중인 백업 용량으로 비용 발생

**백업 관련 정보 항목:**

| 항목 | 설명 |
|------|------|
| **DB 서비스 이름** | 사용자가 지정한 서비스 이름 |
| **Backup 보관일** | 백업 파일의 보관일 |
| **Backup 시작 시간** | 백업이 시작되는 시간 |
| **Backup 데이터 크기** | 완료된 백업 파일의 크기 |
| **마지막 Backup 일자** | 백업이 가장 마지막으로 수행된 일자 |

**복원 방식:**
- 백업 파일을 바탕으로 데이터베이스 복원 기능 제공
- 백업 파일로 복원 시 **신규 VM이 생성**되며, 데이터베이스 서버는 **Recovery 모드**로 복원되어 데이터 조회만 가능
- **시점 복원(Point-In-Time Recovery)** 기능 제공 → 복원 가능한 시간 범위 내에서 **분 단위**까지 원하는 시간대로 데이터 복원 가능

---

### Operation 5: 이벤트 설정 (CDB 모니터링)

Cloud DB for MySQL에 대한 이벤트 설정은 모니터링 전문 서비스인 **Cloud Insight**로 이동하여 Event Rule을 설정하고 통보 알람을 구성한다.

**설정 가능한 주요 Event Rule:**

| metric name | 설명 | Interval | Aggregation |
|------------|------|----------|-------------|
| `mysql_select` | MySQL SELECT qps | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `mysql_insert` | MySQL INSERT qps | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `mysql_update` | MySQL UPDATE qps | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `mysql_delete` | MySQL DELETE qps | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `mysql_replace` | MySQL REPLACE qps | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `mysql_qcache` | MySQL Query Cache ops | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `mysql_call` | MySQL Procedure call aps | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `mysql_totalconn` | MySQL Total Connection | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `mysql_runconn` | MySQL Running Threads | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `mysql_slow` | MySQL Slow Queries | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `cpu_user` | CPU user | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `cpu_sys` | CPU kernel | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `cpu_iowait` | CPU iowait | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `cpu_total` | CPU Total | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `cpu_load_1` | Load avg 1min | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `cpu_load_5` | Load avg 5min | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `cpu_load_15` | Load avg 15min | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `disk_mysql_used` | MySQL Disk Used | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `mem_pct` | Memory used | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `nic_tx` | NIC send | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `nic_rx` | NIC receive | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `swap_pct` | Swap used | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `disk_read` | Disk I/O Read | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `disk_write` | Disk I/O Write | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `mysql_slaverun` | Replication stop | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `mysql_slavedelay` | Replication delay | Min1, Min5, Min30, Hour2, Day1 | AVG, MAX, MIN, SUM, COUNT |
| `aborted_connects` | MySQL 서버 접속 실패 횟수 | Min1, Min5, Min30, Hour2, Day1 | AVG, MIN, MAX, COUNT, SUM |
| `aborted_clients` | 연결을 제대로 닫지 않고 종료한 클라이언트 수 | Min1, Min5, Min30, Hour2, Day1 | AVG, MIN, MAX, COUNT, SUM |

> 임계치를 넘은 이벤트에 대해서 메일과 SMS로 실시간 통보 알람을 받을 수 있다.

---

### Operation 6: DB 엔진 업그레이드

```
업그레이드 순서:
Recovery → Slave → Master (1대씩, 대당 약 1분 내외)

방식: Master → Standby Master로 먼저 전환 후 업그레이드
     → 서비스 중단 최소화
```

**주요 특징:**
- MySQL의 Minor 버전, Major 버전 업그레이드 지원
- 버전 업그레이드는 동일한 서비스 내 모든 DB 서버 버전이 변경
- Master DB는 Standby Master DB로 전환하여 서비스 접근 차단 최소화
- Stand-alone 서버는 업그레이드 진행 동안 DB 접속 불가

---

### Operation 7: DB Config 관리

DB 서버 상세보기 화면 > **DB Config 관리**를 통해 선택한 DB 서버의 설정을 변경할 수 있다.

---

## 주요 Config 파라미터 (MySQL)

| 파라미터 | 기본값 | 설명 |
|---------|--------|------|
| `innodb_buffer_pool_size` | **1073741824** (Byte) | 데이터 파일과 로그 파일 기록 순서 조정 및 디스크 액세스 줄이기 위한 캐시 역할. 보통 메모리의 50~80%로 설정. **수정 후 반드시 DB 재시작 필요** |
| `max_connections` | **3000** | 최대 동시 접속 수. 설정값 초과 시 `Too many connections` 오류 발생. Online 중 설정 및 적용 가능하나 재시작 후 확인 권장 |
| `general_log` | - | MySQL에서 실행되는 전체 쿼리 로그. `general_log` 테이블에 쌓이는 방식. 언제(Time), 누가(User), 어디서(Host) 쿼리를 수행했는지 확인 가능 |
| `wait_timeout` | **28800초 (8시간)** | Connection 연결 유지 시간. DB 접속이 많은 경우 줄여서 리소스 확보 |
| `Connect_timeout` | **10초** | 초기 연결 시 대기 시간 |
| `Autocommit` | **On** | 자동으로 Commit 실행 |
| `character_set_server` | **utf8mb4** | DB 문자셋. 4바이트 문자셋으로 이모지 지원 |
| `long_query_time` | **1초** | Slow Query 기준 시간. 배치, 지표 분석 등 정상적인 쿼리지만 시간이 걸리는 상황에서는 1초에서 늘려준다 |

---

## Lab 11: CDB 생성 및 설정

실습 구성:
1. Cloud DB for MySQL **이벤트 설정** (Cloud Insight 연동)
2. **부하 테스트** 수행
3. **Slave DB 구성** 및 Slave DB 로드밸런서 구성
4. **백업 파일을 이용한 복원**

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

**MSSQL 핵심 기능:**
- 비용이 저렴한 **Standard Edition**을 사용하고도 자동 Fail-over 지원 → 애플리케이션 변경 없이 빠르게 DB 고가용성 지원
- 매일 자동 DB 백업 진행, 데이터 최대 **30일** 보관
- 보관 기간 내 특정 시점으로의 복원 지원
- 설치 후 즉시 DB 모니터링 이용 가능, 메일·SMS 알람 지원
- **1분 단위** 쿼리 레벨 성능 분석 (쿼리 분석 기능)

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

---

### MSSQL Slave DB 상세 (Operation 2: Slave DB 추가)

- Slave DB는 **Log Shipping** 방식으로 생성
- Transaction log 백업을 restore하는 동안 읽기 불가 → 일반 서비스 용도로는 사용 불가
- 매일 주기적인 **BI 및 Batch 작업**에 적합
- 시간 단위로 읽기 가능한 시간을 최대 **20시간**까지 설정 가능
  - 설정한 시간 중에는 DB Restore가 진행되지 않음
  - 모든 시간대 선택은 불가 (최대 20개 시간대 선택)
- 읽기 가능 Slave는 최대 **5대**까지 생성 가능
- 읽기 가능 Slave의 spec을 변경하면 **Principal + Mirror 서버도 함께** 스펙 변경

---

### MSSQL Config Group 관리 (Operation 1: Config Group)

DB Config Group 생성/변경/삭제가 가능하다. Config 적용 시 일부 항목은 재시작 이후에 적용된다.

**주요 Config 파라미터 (MSSQL):**

| 파라미터 | 기본값 | 설명 |
|---------|--------|------|
| `user connections` | **32767** | 최대 동시 접속 수 (`sp_configure`로 설정) |
| `remote access` | **0 (허용)** | SQL Server 인스턴스가 실행되는 로컬 또는 원격 서버에서 저장 프로시저 실행 제어. 기본값 0이며 원격 서버에서 로컬 저장 프로시저 실행, 또는 로컬 서버에서 원격 저장 프로시저 실행 권한 부여 |
| `remote query timeout` | **600초 (10분)** | Connection 연결 유지 시간 |
| `max worker threads` | 128 ~ 65535 | 최대 워커 스레드 수 |
| `network packet size` | 4096 ~ 32767 | 네트워크 패킷 사이즈 |

---

### MSSQL 이벤트 설정 (Operation 3: 이벤트 설정)

Cloud Insight를 통해 알람 항목과 임계치를 설정하여 임계치를 넘은 이벤트에 대해 메일과 SMS로 실시간 통보 설정이 가능하다.

**이벤트 유형 예시:**

| 이벤트 유형 | 이벤트 이름 | 설명 |
|------------|------------|------|
| MSSQL Server | Restart | server restart completed |
| MSSQL Server | Backup | full backup completed (master/media) |
| MSSQL Server | Account | user account created |

---

### MSSQL 쿼리 분석 (Operation 4: 쿼리 분석)

```
[버블 차트 형태 제공 - 1분 단위]
X축: 해당 쿼리의 일별 쿼리 수행 횟수 합계
Y축: 해당 쿼리의 일별 CPU 소모량의 합계
버블 크기: 해당 쿼리의 메모리 읽기 수

→ 버블을 클릭하면 어떤 쿼리인지 하단에서 확인 가능
```

쿼리 수행 횟수 대비 CPU 소모량과 메모리 읽기 수의 상관관계 및 분포를 한눈에 파악할 수 있어 성능 병목 쿼리를 빠르게 식별할 수 있다.

---

## 📝 시험 대비 체크리스트

**MySQL:**
- [ ] DB 엔진 버전: **8.0.34, 8.0.36, 8.0.40, 8.0.42**
- [ ] 최대 스토리지: **6TB** (자동 확장, 기본 10GB)
- [ ] 최대 스펙: **32 vCPU, 256GB**
- [ ] Fail-over: 별도 모니터 서버, **1분 단위** 체크
- [ ] Read Replica: 최대 **10대**
- [ ] 업그레이드 순서: **Recovery → Slave → Master**
- [ ] 백업: 하루 1회, 최소 1일 ~ 최대 **30일** 보관, 미설정 시 **01:00** 실행
- [ ] 시점 복원: **분 단위** 지원
- [ ] `innodb_buffer_pool_size` 기본값: **1073741824 (Byte)**, 수정 후 **재시작 필요**
- [ ] `max_connections` 기본값: **3000**
- [ ] `wait_timeout` 기본값: **28800초 (8시간)**
- [ ] `Connect_timeout` 기본값: **10초**
- [ ] `character_set_server` 기본값: **utf8mb4** (4바이트, 이모지 지원)
- [ ] `long_query_time` 기본값: **1초**
- [ ] Replication 지연 알람: Event 메뉴에서 등록 (항목: `Replication` / 세부: `Delay`)

**MSSQL:**
- [ ] 최대 스토리지: **2TB**
- [ ] 최대 스펙: **24 vCPU, 128GB**
- [ ] Slave 방식: **Log Shipping** (실시간 읽기 **불가**)
- [ ] MSSQL Slave 용도: **BI/Batch 전용**
- [ ] 읽기 가능 Slave: 최대 **5대**
- [ ] 읽기 가능 시간: 하루 최대 **20시간** (최대 20개 시간대)
- [ ] Slave spec 변경 시 **Principal + Mirror 서버도 함께 변경**
- [ ] `user connections` 기본값: **32767**
- [ ] `remote access` 기본값: **0 (허용)**
- [ ] `remote query timeout`: **600초 (10분)**
- [ ] Standard Edition도 **자동 Fail-over** 지원
- [ ] 쿼리 분석: **버블 차트**, X축=수행 횟수, Y축=CPU 소모량, 버블 크기=메모리 읽기 수

---

## 🧠 암기 핵심 문장

> "**MySQL Read Replica = 실시간 10대 / MSSQL Slave = Log Shipping, BI/Batch, 5대**"  
> "**MySQL Fail-over = 1분 단위 모니터링 / MSSQL = Standard도 자동 Fail-over**"  
> "**DB Replication = SPOF 해결 + 읽기/쓰기 분리 + 부하 분산**"  
> "**MySQL 백업 = 하루 1회, 최대 30일, 기본 01:00, 분 단위 시점 복원**"

---

*다음 챕터: `20_데이터베이스_비교.md` → PostgreSQL, Redis, MongoDB + DB 종합 비교*
