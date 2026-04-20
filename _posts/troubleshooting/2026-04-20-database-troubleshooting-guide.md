---
title: "데이터베이스 트러블슈팅 실무 가이드 (MySQL / PostgreSQL / Redis)"
date: 2026-04-20 04:00:00 +0900
categories: [Tips, Troubleshooting]
tags: [mysql, postgresql, redis, database, troubleshooting, slow-query, lock, replication]
description: "MySQL, PostgreSQL, Redis 장애 진단 및 대응 실무 가이드 - 락/데드락 분석, 슬로우 쿼리, 복제 점검"
---

> 주요 데이터베이스 장애 진단 및 대응 실무 가이드입니다.

## 목차

1. [공통 점검 흐름](#1-공통-점검-흐름)
2. [MySQL / MariaDB 점검](#2-mysql--mariadb-점검)
3. [PostgreSQL 점검](#3-postgresql-점검)
4. [Redis 점검](#4-redis-점검)
5. [공통 장애 시나리오](#5-공통-장애-시나리오)

---

## 1. 공통 점검 흐름

### 1.1 DB 장애 표준 대응 순서

```
1. 연결 가능 여부 확인       → 포트 연결, 인증
2. 프로세스/서비스 상태       → systemctl, ps
3. 리소스 사용량              → CPU, 메모리, 디스크 I/O
4. 활성 연결 / 세션          → 동시 접속자 수, idle 세션
5. 슬로우 쿼리 / 락           → 락 대기, 장기 실행 쿼리
6. 에러 로그                  → DB 에러 로그 확인
7. 디스크 / 데이터 파일       → 용량, 권한, 무결성
```

### 1.2 1차 점검 우선순위

| 우선순위 | 점검 항목 | 명령어 예시 |
|---------|---------|-----------|
| 1 | DB 살아있는지 | `systemctl status mysql` |
| 2 | 포트 LISTEN | `ss -tlnp \| grep 3306` |
| 3 | 연결 가능한지 | `mysql -e "SELECT 1"` |
| 4 | 디스크 가득찼는지 | `df -h` |
| 5 | 메모리/CPU | `top`, `free -h` |
| 6 | 활성 쿼리 | `SHOW PROCESSLIST` |
| 7 | 에러 로그 | DB별 로그 파일 |

---

## 2. MySQL / MariaDB 점검

### 2.1 기본 정보

| 항목 | 경로 |
|------|------|
| 서비스명 | `mysql`, `mysqld`, `mariadb` |
| 설정 파일 | `/etc/mysql/my.cnf`, `/etc/my.cnf` |
| 데이터 디렉토리 | `/var/lib/mysql/` |
| 에러 로그 | `/var/log/mysql/error.log` (Ubuntu), `/var/log/mysqld.log` (RHEL) |
| 슬로우 쿼리 로그 | `/var/log/mysql/slow.log` |
| 기본 포트 | 3306 |

### 2.2 기본 점검 명령어

```bash
systemctl status mysql
sudo ss -tlnp | grep 3306
mysql --version
mysql -u root -p
mysql -h <호스트> -u <사용자> -p -P 3306
```

### 2.3 주요 쿼리 (DB 내부에서)

```sql
-- 현재 연결 수
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Threads_running';
SHOW VARIABLES LIKE 'max_connections';

-- 활성 프로세스/쿼리 목록
SHOW FULL PROCESSLIST;

-- 5초 이상 실행 중인 쿼리만
SELECT id, user, host, db, command, time, state, info
FROM information_schema.processlist
WHERE time > 5 AND command != 'Sleep'
ORDER BY time DESC;

-- 데이터베이스 크기
SELECT table_schema AS 'DB',
       ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.tables
GROUP BY table_schema
ORDER BY 2 DESC;

-- InnoDB 상태
SHOW ENGINE INNODB STATUS\G
```

### 2.4 락 / 데드락 분석

```sql
-- 현재 락 대기 중인 트랜잭션 (MySQL 8.0+)
SELECT * FROM performance_schema.data_lock_waits\G

-- 누가 누구를 기다리는지
SELECT
  r.trx_id waiting_trx_id,
  r.trx_mysql_thread_id waiting_thread,
  r.trx_query waiting_query,
  b.trx_id blocking_trx_id,
  b.trx_mysql_thread_id blocking_thread,
  b.trx_query blocking_query
FROM performance_schema.data_lock_waits w
JOIN information_schema.innodb_trx b ON b.trx_id = w.blocking_engine_transaction_id
JOIN information_schema.innodb_trx r ON r.trx_id = w.requesting_engine_transaction_id;

-- 데드락 이력
SHOW ENGINE INNODB STATUS\G
-- → "LATEST DETECTED DEADLOCK" 섹션

-- 강제 종료
KILL <process_id>;
```

### 2.5 슬로우 쿼리 분석

```sql
-- 슬로우 쿼리 로그 활성화
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;        -- 1초 이상
```

```bash
mysqldumpslow -s t -t 10 /var/log/mysql/slow.log     # 시간 기준 TOP 10
pt-query-digest /var/log/mysql/slow.log               # Percona Toolkit (권장)
```

### 2.6 인덱스 / 실행 계획

```sql
EXPLAIN SELECT * FROM users WHERE email = 'a@b.com';
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'a@b.com';  -- MySQL 8.0+
SHOW INDEX FROM users;
```

### 2.7 복제 (Replication) 점검

```sql
SHOW MASTER STATUS;
SHOW SLAVE STATUS\G

-- 주요 확인 항목
-- Slave_IO_Running:        Yes      ← 두 개 다 Yes 여야 정상
-- Slave_SQL_Running:       Yes
-- Seconds_Behind_Master:   0        ← 지연 시간 (초)
-- Last_IO_Error:                    ← 에러 메시지

-- 복제 재시작
STOP SLAVE;
START SLAVE;
```

### 2.8 자주 보는 MySQL 에러

| 에러 | 원인 | 조치 |
|------|------|------|
| `Too many connections` | 최대 연결 수 도달 | `max_connections` 증가, idle 세션 정리 |
| `Lock wait timeout exceeded` | 락 대기 시간 초과 | `innodb_lock_wait_timeout` 조정 |
| `Deadlock found when trying to get lock` | 데드락 | `SHOW ENGINE INNODB STATUS`로 분석 |
| `Disk full` | 디스크 가득 | 용량 확보, 바이너리 로그 정리 |
| `MySQL server has gone away` | 연결 끊김 | `wait_timeout`, `max_allowed_packet` 확인 |
| `Access denied for user` | 권한 문제 | GRANT 확인 |

### 2.9 백업 / 복구

```bash
# 논리 백업
mysqldump -u root -p --single-transaction --routines --triggers mydb > mydb.sql
mysqldump -u root -p mydb | gzip > mydb-$(date +%Y%m%d).sql.gz

# 복구
mysql -u root -p mydb < mydb.sql

# 바이너리 로그 정리 (디스크 확보)
PURGE BINARY LOGS BEFORE NOW() - INTERVAL 7 DAY;
```

---

## 3. PostgreSQL 점검

### 3.1 기본 정보

| 항목 | 경로 |
|------|------|
| 설정 파일 | `/etc/postgresql/<ver>/main/postgresql.conf` |
| 인증 설정 | `pg_hba.conf` |
| 로그 | `/var/log/postgresql/postgresql-<ver>-main.log` |
| 기본 포트 | 5432 |

### 3.2 기본 점검 명령어

```bash
systemctl status postgresql
sudo ss -tlnp | grep 5432
sudo -u postgres psql -c "SELECT version();"
sudo -u postgres psql -c "SHOW data_directory;"
```

### 3.3 주요 쿼리 (psql 내부)

```sql
-- 현재 연결 수
SELECT count(*) FROM pg_stat_activity;
SHOW max_connections;

-- 5초 이상 실행 중인 쿼리
SELECT pid, now() - query_start AS duration, query, state
FROM pg_stat_activity
WHERE state = 'active'
  AND now() - query_start > interval '5 seconds'
ORDER BY duration DESC;

-- DB별 크기
SELECT datname, pg_size_pretty(pg_database_size(datname))
FROM pg_database
ORDER BY pg_database_size(datname) DESC;

-- 테이블별 크기
SELECT schemaname, tablename,
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS total_size
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 20;
```

### 3.4 락 / 블로킹

```sql
-- 락 대기 중인 쿼리 (블로킹 트리)
SELECT
  blocked.pid AS blocked_pid,
  blocked.query AS blocked_query,
  blocking.pid AS blocking_pid,
  blocking.query AS blocking_query
FROM pg_stat_activity blocked
JOIN pg_stat_activity blocking ON blocking.pid = ANY(pg_blocking_pids(blocked.pid));

-- 강제 종료
SELECT pg_cancel_backend(<pid>);    -- 쿼리만 취소
SELECT pg_terminate_backend(<pid>); -- 세션 강제 종료
```

### 3.5 슬로우 쿼리 / 통계

```sql
-- pg_stat_statements 활성화 필요 (postgresql.conf)
-- shared_preload_libraries = 'pg_stat_statements'
CREATE EXTENSION pg_stat_statements;

-- 가장 느린 쿼리
SELECT query, calls, total_exec_time, mean_exec_time, rows
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 20;
```

### 3.6 VACUUM / 통계 정보

PostgreSQL은 MVCC 구조라 주기적 VACUUM이 필요합니다.

```sql
-- Dead 튜플 비율 높은 테이블 (VACUUM 필요)
SELECT schemaname, relname,
       n_live_tup, n_dead_tup,
       ROUND(n_dead_tup * 100.0 / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_ratio
FROM pg_stat_user_tables
WHERE n_live_tup > 1000
ORDER BY dead_ratio DESC NULLS LAST
LIMIT 20;

VACUUM ANALYZE table_name;
VACUUM FULL table_name;            -- 락 발생, 운영 중 금지
```

### 3.7 자주 보는 PostgreSQL 에러

| 에러 | 원인 | 조치 |
|------|------|------|
| `FATAL: too many connections` | 최대 연결 도달 | `max_connections` 증가, pgbouncer |
| `FATAL: password authentication failed` | 인증 실패 | `pg_hba.conf` 확인 |
| `deadlock detected` | 데드락 | 트랜잭션 순서 변경 |
| `disk full` | 디스크 가득 | WAL 정리, 데이터 정리 |

### 3.8 백업 / 복구

```bash
pg_dump -U postgres -d mydb -Fc > mydb.dump      # 커스텀 포맷 (권장)
pg_dumpall -U postgres > all.sql
pg_restore -U postgres -d mydb mydb.dump
```

---

## 4. Redis 점검

### 4.1 기본 정보

| 항목 | 경로 |
|------|------|
| 설정 파일 | `/etc/redis/redis.conf` |
| 데이터 디렉토리 | `/var/lib/redis/` |
| 로그 | `/var/log/redis/redis-server.log` |
| 기본 포트 | 6379 |

### 4.2 기본 명령어

```bash
systemctl status redis
sudo ss -tlnp | grep 6379
redis-cli ping
redis-cli info
```

### 4.3 INFO 분석

```bash
redis-cli info server         # 서버 정보
redis-cli info clients        # 연결된 클라이언트
redis-cli info memory         # 메모리 사용량
redis-cli info replication    # 복제 상태
redis-cli info keyspace       # 키 개수
```

**주요 INFO 항목:**

| 항목 | 의미 | 주의점 |
|------|------|-------|
| `connected_clients` | 현재 연결 수 | maxclients 대비 |
| `used_memory_human` | 사용 메모리 | maxmemory 대비 |
| `mem_fragmentation_ratio` | 메모리 단편화 | 1.0~1.5 정상 |
| `evicted_keys` | 메모리 부족으로 삭제된 키 | 증가하면 메모리 부족 |
| `keyspace_hits` / `keyspace_misses` | 캐시 hit/miss | hit 비율 확인 |
| `instantaneous_ops_per_sec` | 초당 명령 수 | 부하 확인 |

### 4.4 자주 쓰는 진단 명령어

```bash
redis-cli config get maxmemory
redis-cli config get maxclients
redis-cli dbsize                            # 키 개수

# 슬로우 로그
redis-cli slowlog get 10
redis-cli config set slowlog-log-slower-than 10000   # 10ms 이상

redis-cli client list
redis-cli monitor                           # 실시간 명령어 (운영 환경 부하 주의)

# 큰 키 찾기 (SCAN 기반, 안전)
redis-cli --bigkeys
```

### 4.5 키 분석

```bash
# KEYS는 운영 환경에서 절대 사용 금지 (블로킹)
# 대신 SCAN 사용
redis-cli --scan --pattern 'user:*' | head
```

### 4.6 복제 점검

```bash
# Master에서
redis-cli info replication
# role:master, connected_slaves:1

# Slave에서
redis-cli info replication
# master_link_status:up, master_last_io_seconds_ago:1
```

### 4.7 자주 보는 Redis 에러

| 에러 | 원인 | 조치 |
|------|------|------|
| `OOM command not allowed when used memory > 'maxmemory'` | 메모리 한계 초과 | `maxmemory` 증가, eviction 정책 검토 |
| `READONLY You can't write against a read only replica` | Slave에 쓰기 시도 | Master로 연결 |
| `MAX number of clients reached` | 클라이언트 한계 | `maxclients` 증가, 연결 누수 점검 |

### 4.8 Eviction 정책

```bash
redis-cli config get maxmemory-policy
# 캐시 용도면 allkeys-lru 또는 allkeys-lfu 권장
redis-cli config set maxmemory-policy allkeys-lru
```

### 4.9 백업 / 복구

```bash
redis-cli bgsave                # 비동기 RDB 저장 (권장)
redis-cli bgrewriteaof          # AOF 재작성
cp /var/lib/redis/dump.rdb /backup/dump-$(date +%Y%m%d).rdb

# 복구: Redis 정지 후 dump.rdb 교체
sudo systemctl stop redis
sudo cp /backup/dump.rdb /var/lib/redis/
sudo chown redis:redis /var/lib/redis/dump.rdb
sudo systemctl start redis
```

---

## 5. 공통 장애 시나리오

### 시나리오 1: "DB가 너무 느려요"

```bash
top && iostat -x 1 5

# 활성 쿼리 확인
# MySQL: SHOW FULL PROCESSLIST;
# PostgreSQL: SELECT * FROM pg_stat_activity WHERE state != 'idle';
# Redis: redis-cli slowlog get 20

# 락 대기 확인
# MySQL: SHOW ENGINE INNODB STATUS;
# PostgreSQL: pg_blocking_pids 쿼리
```

### 시나리오 2: "DB 연결이 안 돼요"

```bash
ps aux | grep mysql
sudo ss -tlnp | grep 3306
mysql -u root -p
nc -zv <DB호스트> 3306
sudo iptables -L -n

# 연결 한계 도달 여부
# MySQL: SHOW STATUS LIKE 'Threads_connected'; vs max_connections
# PostgreSQL: SELECT count(*) FROM pg_stat_activity; vs max_connections
# Redis: redis-cli info clients
```

### 시나리오 3: "DB 디스크가 가득 찼어요"

```bash
df -h
sudo du -sh /var/lib/mysql/

# MySQL: 바이너리 로그 정리
# PURGE BINARY LOGS BEFORE NOW() - INTERVAL 7 DAY;
# 슬로우 쿼리 로그 비우기
# > /var/log/mysql/slow.log
```

### 시나리오 4: "DB가 갑자기 죽었어요"

```bash
sudo journalctl -u mysql --since "1 hour ago"
sudo tail -200 /var/log/mysql/error.log
dmesg | grep -i "killed process\|oom"   # OOM Killer 확인
sudo systemctl start mysql
```

### 시나리오 5: "특정 쿼리가 락에 걸려요"

```sql
-- MySQL
SELECT * FROM performance_schema.data_lock_waits\G
SHOW ENGINE INNODB STATUS\G
KILL <process_id>;
```

```sql
-- PostgreSQL
SELECT pid, pg_blocking_pids(pid) AS blocked_by, query
FROM pg_stat_activity
WHERE pg_blocking_pids(pid)::text != '{}';

SELECT pg_terminate_backend(<pid>);
```

---

## 6. 운영 체크리스트

### 일상 점검
- [ ] 서비스 살아있음 (`systemctl status`)
- [ ] 연결 수 정상 범위 (`max_connections`의 70% 미만)
- [ ] 디스크 사용률 80% 미만
- [ ] 슬로우 쿼리 급증 없음
- [ ] 복제 지연 1초 이내 (해당 시)

### 변경 작업 전
- [ ] 백업 확보
- [ ] DDL은 가급적 트래픽 적은 시간에
- [ ] MySQL: pt-online-schema-change 활용 (대용량 ALTER)
- [ ] 롤백 계획 수립

---

## 7. 운영 시 주의사항

- **운영 DB에서 `SELECT *` 풀 스캔 금지** — 슬로우 쿼리 + 메모리 폭주
- **`KEYS *` (Redis), `LIKE '%text%'` 풀 스캔 금지**
- **`KILL`은 신중히** — 트랜잭션 롤백 시간 동안 락 유지될 수 있음
- **`VACUUM FULL` (PostgreSQL)은 ACCESS EXCLUSIVE 락** — 운영 중 금지
- **백업 없이 DROP/TRUNCATE 금지**
- **운영 DB 직접 수정 시 반드시 트랜잭션 사용** (`BEGIN; ... COMMIT;`)
