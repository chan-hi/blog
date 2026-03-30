---
title: "NCP DMS 마이그레이션 트러블슈팅 총정리"
date: 2026-03-30 10:00:00 +0900
categories: [NCP, Migration]
tags: [ncp, dms, mysql, troubleshooting, mysql-8.4, debugging]
description: "NCP DMS로 MySQL 마이그레이션 중 발생한 에러들과 근본 원인 분석 — MySQL 8.4 미지원 버그 포함"
---

## 환경
- 소스 DB: 로컬 Ubuntu (Linux) MySQL
- 타겟 DB: NCP Cloud DB for MySQL 8.4.6
- 도구: NCP DMS (Database Migration Service)

---

## 최종 결론

> **NCP DMS가 MySQL 8.4를 완전히 지원하지 않는 버그**
> DMS 내부에서 MySQL 8.4에서 제거된 `SHOW SLAVE STATUS` 명령을 사용하여 에러 발생.
> 소스 DB를 **MySQL 8.0**으로 맞춰야 정상 동작함.

---

## 과정 및 발생 에러 정리

### 1단계: MySQL 서버 설치
- 로컬에 mysql-client만 설치되어 있고 **mysql-server가 없었음**
- 해결: `sudo apt install -y mysql-server` 로 8.0 설치

---

### 2단계: DMS 첫 연결 시도 → 에러 1
```
mysqldump: Couldn't execute 'SHOW BINARY LOG STATUS':
You have an error in your SQL syntax (1064)
```
**원인:** 로컬 MySQL이 8.0인데, DMS가 MySQL 8.2+ 전용 문법인 `SHOW BINARY LOG STATUS`를 사용
**판단:** NCP Cloud DB가 MySQL 8.4.6이므로 소스도 8.4로 맞춰야 한다고 판단 → MySQL 8.4 업그레이드 시도

---

### 3단계: MySQL 8.4 업그레이드 시도 → GPG 키 문제
```
EXPKEYSIG B7B3B788A8D3785C MySQL Release Engineering
The repository is not signed.
```
**원인:** MySQL 공식 APT 저장소의 GPG 키가 **2025-10-22 만료**됨
**해결:** MySQL 8.4 deb 패키지 직접 다운로드 후 설치

---

### 4단계: MySQL 8.4 설치 완료 후 → 에러 2
```
ERROR 1524 (HY000): Plugin 'mysql_native_password' is not loaded
```
**원인:** MySQL 8.4에서 `mysql_native_password` 플러그인이 기본 비활성화
**해결:** `mysqld.cnf`에 `mysql_native_password=ON` 추가 후 재시작

---

### 5단계: DMS 재연결 시도 → 에러 3
```
Authentication plugin 'caching_sha2_password' reported error:
Authentication requires secure connection.
```
**원인:** `caching_sha2_password`는 SSL 연결 필요, DMS는 SSL 없이 접속 시도
**해결:** dms_user 인증 방식을 `mysql_native_password`로 변경

---

### 6단계: DMS 재연결 시도 → 에러 4 (핵심)

**NCP DMS 콘솔 에러 로그:**
```
exporting error: ERROR 1064 (42000) at line 1: You have an error in your SQL syntax;
check the manual that corresponds to your MySQL server version for the right syntax
to use near '' at line 1

importing error: ERROR 1064 (42000) at line 1: You have an error in your SQL syntax;
check the manual that corresponds to your MySQL server version for the right syntax
to use near '' at line 1
```

**원인 분석 과정:**

DMS 에러 로그만으로는 `near ''` 가 어떤 쿼리에서 발생하는지 특정 불가.
MySQL general log를 활성화하여 DMS가 소스 DB에 실행하는 모든 쿼리를 실시간 추적.

```sql
-- general log 활성화
SET GLOBAL general_log = 'ON';
SET GLOBAL general_log_file='/var/log/mysql/general.log';
```

```bash
-- DMS 재실행 후 로그 확인
tail -f /var/log/mysql/general.log
```

로그에서 DMS가 실행하는 쿼리 순서를 확인:

```
49 Query  FLUSH /*!40101 LOCAL */ TABLES WITH READ LOCK
49 Query  SHOW BINARY LOG STATUS     ← 정상 동작 (8.4 지원)
49 Query  SHOW SLAVE STATUS          ← 8.4에서 제거된 명령어
```

`SHOW SLAVE STATUS`를 직접 실행해서 에러 재현:

```sql
mysql> SHOW SLAVE STATUS;
ERROR 1064 (42000): You have an error in your SQL syntax...
near 'SLAVE STATUS' at line 1
```

`near ''` 에러의 정체는 MySQL 8.4가 `SLAVE` 키워드 자체를 인식 못해 빈 문자열로 파싱한 것.
DMS가 내부적으로 `SHOW SLAVE STATUS`를 사용하고 있어 **NCP DMS가 MySQL 8.4 미지원**임을 확인.

```sql
-- general log에서 확인된 DMS 실행 쿼리 목록
SHOW VARIABLES LIKE 'gtid_mode'
FLUSH LOCAL TABLES WITH READ LOCK
SHOW BINARY LOG STATUS        ← MySQL 8.4에서 정상 동작
SHOW SLAVE STATUS             ← MySQL 8.4에서 완전 제거된 명령어!
```

---

### 7단계: MySQL 8.0으로 다운그레이드
MySQL 8.4 제거 → MySQL 8.0 재설치 → 데이터 복원 → DMS 설정 재구성

---

## 최종 로컬 MySQL 설정 (정상 동작)

### 서버 정보

| 항목 | 값 |
|------|-----|
| MySQL 버전 | 8.0.45 |
| bind-address | 0.0.0.0 (외부 접속 허용) |
| 포트 | 3306 |

### Binary Log 설정 (DMS 필수)

| 항목 | 값 |
|------|-----|
| log_bin | ON |
| binlog_format | ROW |
| binlog_row_image | FULL |
| server_id | 1 |
| binlog_expire_logs_seconds | 2592000 (30일) |

### DMS 계정

| 항목 | 값 |
|------|-----|
| 사용자 | dms_user |
| 인증 방식 | mysql_native_password |
| 권한 | SELECT, RELOAD, PROCESS, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT |

### 테스트 데이터

| 테이블 | 건수 |
|--------|------|
| testdb.users | 3건 |
| testdb.orders | 4건 |

---

## NCP DMS MySQL 버전 호환성 정리

| MySQL 버전 | DMS 지원 여부 | 비고 |
|-----------|-------------|------|
| 8.0.x | ✅ 정상 동작 | 권장 버전 |
| 8.2.x | ⚠️ 미확인 | |
| 8.4.x | ❌ 미지원 | `SHOW SLAVE STATUS` 제거로 에러 발생 |

---

## MySQL 버전별 주요 변경사항 (DMS 관련)

| 명령어 | 8.0 | 8.2 | 8.4 |
|--------|-----|-----|-----|
| `SHOW MASTER STATUS` | ✅ | ⚠️ deprecated | ❌ 제거 |
| `SHOW BINARY LOG STATUS` | ❌ | ✅ 추가 | ✅ |
| `SHOW SLAVE STATUS` | ✅ | ⚠️ deprecated | ❌ 제거 |
| `SHOW REPLICA STATUS` | ❌ | ✅ 추가 | ✅ |
| `mysql_native_password` | ✅ 기본 | ⚠️ deprecated | ⚠️ 비활성화 |

---

## 디버깅에 사용한 명령어

### General Log 활성화 (DMS 쿼리 추적)
```sql
SET GLOBAL general_log = 'ON';
SET GLOBAL general_log_file='/var/log/mysql/general.log';
```
```bash
tail -f /var/log/mysql/general.log
```

### DMS 필수 조건 한번에 확인
```sql
SHOW VARIABLES LIKE 'log_bin';
SHOW VARIABLES LIKE 'binlog_format';
SHOW VARIABLES LIKE 'binlog_row_image';
SHOW VARIABLES LIKE 'server_id';
SHOW VARIABLES LIKE 'bind_address';
SHOW VARIABLES LIKE 'binlog_expire_logs_seconds';
SHOW MASTER STATUS;
SHOW GRANTS FOR 'dms_user'@'%';
```

---

## 참고
- [NCP DMS 공식 문서](https://guide.ncloud-docs.com/docs/dms-overview)
- [NCP DMS 트러블슈팅](https://guide.ncloud-docs.com/docs/dms-troubleshot-common)
