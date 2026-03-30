---
title: "NCP DMS 마이그레이션 가이드 (Local MySQL → NCP Cloud DB for MySQL)"
date: 2026-03-30 09:00:00 +0900
categories: [NCP, Migration]
tags: [ncp, dms, mysql, migration, cloud-db, database]
description: "로컬 Ubuntu MySQL 8.0에서 NCP Cloud DB for MySQL로 DMS를 활용해 마이그레이션하는 단계별 가이드"
---

## 환경 정보
- 로컬 OS: Ubuntu (Linux)
- MySQL 버전: 8.0.44 (이미 설치됨)
- 목적지: NCP Cloud DB for MySQL

---

## 1단계: 로컬 MySQL 서비스 시작 및 확인

```bash
# MySQL 서비스 시작
sudo service mysql start

# 상태 확인
sudo service mysql status

# root로 접속 테스트
sudo mysql -u root
```

---

## 2단계: MySQL root 비밀번호 설정

MySQL에 접속 후 아래 명령 실행:

```sql
-- 비밀번호 설정 (DMS가 접근할 수 있도록 password 방식으로 변경)
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YourPassword123!';
FLUSH PRIVILEGES;
EXIT;
```

비밀번호 설정 후 재접속 테스트:

```bash
mysql -u root -p
```

---

## 3단계: DMS용 전용 계정 생성

DMS는 외부에서 접근하므로 별도 계정 생성 권장:

```sql
-- DMS 전용 계정 생성 (% = 모든 IP 허용)
CREATE USER 'dms_user'@'%' IDENTIFIED WITH mysql_native_password BY 'DmsPassword123!';

-- 필요한 권한 부여 (DMS 소스 DB 요구 권한)
GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'dms_user'@'%';

FLUSH PRIVILEGES;

-- 권한 확인
SHOW GRANTS FOR 'dms_user'@'%';
```

---

## 4단계: 테스트용 샘플 DB 및 데이터 생성

```sql
-- 테스트 DB 생성
CREATE DATABASE testdb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE testdb;

-- 테스트 테이블 생성
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(200) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    product VARCHAR(200),
    amount DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- 샘플 데이터 삽입
INSERT INTO users (name, email) VALUES
    ('홍길동', 'hong@test.com'),
    ('김철수', 'kim@test.com'),
    ('이영희', 'lee@test.com');

INSERT INTO orders (user_id, product, amount) VALUES
    (1, '노트북', 1200000),
    (1, '마우스', 35000),
    (2, '키보드', 85000),
    (3, '모니터', 450000);

-- 데이터 확인
SELECT * FROM users;
SELECT * FROM orders;
```

---

## 5단계: MySQL 외부 접속 허용 설정

### my.cnf 설정 변경

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

아래 항목을 찾아서 수정:

```ini
# 기존
bind-address = 127.0.0.1

# 변경 (모든 IP 허용)
bind-address = 0.0.0.0
```

### Binary Log 활성화 (CDC 방식 DMS에 필수)

같은 파일 `[mysqld]` 섹션에 아래 내용 추가:

```ini
server-id        = 1
log_bin          = /var/log/mysql/mysql-bin.log
binlog_format    = ROW
binlog_row_image = FULL
expire_logs_days = 7
```

### MySQL 재시작

```bash
sudo service mysql restart

# 3306 포트 확인
sudo ss -tlnp | grep 3306
```

---

## 6단계: 방화벽 설정

```bash
# 3306 포트 허용
sudo ufw allow 3306/tcp
sudo ufw status
```

> NCP DMS 에이전트 IP 대역만 허용하면 더 안전합니다. NCP 콘솔에서 DMS IP 확인 후 설정하세요.

---

## 7단계: Binary Log 및 설정 확인

```sql
-- Binary log 활성화 확인
SHOW VARIABLES LIKE 'log_bin';
SHOW VARIABLES LIKE 'binlog_format';
SHOW VARIABLES LIKE 'server_id';
SHOW VARIABLES LIKE 'bind_address';

-- 현재 binlog 위치 확인 (DMS 설정 시 필요)
SHOW MASTER STATUS;
```

---

## 8단계: 로컬 Public IP 확인

NCP DMS 소스 DB 설정 시 필요:

```bash
curl ifconfig.me
```

---

## 9단계: NCP 콘솔에서 DMS 설정

### 9-1. NCP Cloud DB for MySQL 생성
1. NCP 콘솔 → **Cloud DB for MySQL** → 서버 생성
2. DB 버전: MySQL 8.0 (소스와 동일하게)
3. 접속 포트, 마스터 계정 설정

### 9-2. DMS 태스크 생성
1. NCP 콘솔 → **DMS** → 마이그레이션 태스크 생성

### 9-3. 소스 DB 설정 (로컬 MySQL)

| 항목 | 값 |
|------|-----|
| DB 엔진 | MySQL |
| 호스트 | 로컬 서버의 Public IP |
| 포트 | 3306 |
| 사용자 | dms_user |
| 비밀번호 | DmsPassword123! |
| DB명 | testdb |

### 9-4. 마이그레이션 방식 선택

| 방식 | 설명 |
|------|------|
| Full Load | 전체 데이터 일회성 복사 (서비스 중단 필요) |
| CDC | Binary log 기반 실시간 복제 (무중단) |
| Full Load + CDC | 초기 복사 후 변경분 동기화 **(권장)** |

---

## 10단계: 마이그레이션 검증

```sql
-- 소스 DB 레코드 수 확인
SELECT COUNT(*) FROM testdb.users;
SELECT COUNT(*) FROM testdb.orders;

-- CDC 테스트: 데이터 추가 후 NCP Cloud DB에서도 동기화 확인
INSERT INTO testdb.users (name, email) VALUES ('신규유저', 'new@test.com');
```

NCP Cloud DB에서 동일하게 조회하여 동기화 여부 검증.

---

## 트러블슈팅

### MySQL 접속 불가
```bash
sudo tail -f /var/log/mysql/error.log
```

### 원격 접속 불가
```bash
# 포트 열려있는지 확인
telnet <로컬IP> 3306
```

### DMS 권한 오류
```sql
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'dms_user'@'%';
FLUSH PRIVILEGES;
```

### Binary Log 미활성화
```sql
SHOW VARIABLES LIKE '%binlog%';
-- log_bin = ON 이어야 함
```

---

## 참고 링크
- [NCP DMS 공식 문서](https://guide.ncloud-docs.com/docs/dms-overview)
- [NCP Cloud DB for MySQL](https://guide.ncloud-docs.com/docs/clouddbformysql-overview)
