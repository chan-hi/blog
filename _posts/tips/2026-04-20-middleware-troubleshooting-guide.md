---
title: "미들웨어 트러블슈팅 실무 가이드 (Nginx / Apache / Tomcat)"
date: 2026-04-20 03:00:00 +0900
categories: [Tips, Troubleshooting]
tags: [nginx, apache, tomcat, middleware, troubleshooting, 502, jvm, thread-dump]
description: "웹서버(Nginx, Apache) 및 WAS(Tomcat) 장애 대응 실무 가이드 - 에러 로그 분석, HTTP 상태코드 대응, JVM 진단"
---

> 웹서버(Nginx, Apache) 및 WAS(Tomcat) 장애 대응 실무 가이드입니다.

## 목차

1. [공통 점검 흐름](#1-공통-점검-흐름)
2. [Nginx 점검](#2-nginx-점검)
3. [Apache HTTPD 점검](#3-apache-httpd-점검)
4. [Tomcat 점검](#4-tomcat-점검)
5. [HTTP 상태 코드별 대응](#5-http-상태-코드별-대응)
6. [실전 시나리오](#6-실전-시나리오)

---

## 1. 공통 점검 흐름

### 1.1 미들웨어 장애 표준 대응 순서

```
1. 서비스 응답 확인         → curl로 증상 재현
2. 프로세스 생존 확인       → ps, systemctl status
3. 포트 LISTEN 확인         → ss -tlnp
4. 설정 파일 문법 검증       → nginx -t, apachectl -t
5. 에러 로그 확인           → error_log, catalina.out
6. 액세스 로그 패턴 분석     → access_log
7. 커넥션/스레드 상태        → ss, jstack
8. 리소스 확인              → top, free, df
```

### 1.2 1차 점검 원샷 명령어

```bash
systemctl status nginx && \
sudo ss -tlnp | grep -E ':(80|443)' && \
ps aux | grep -E "nginx|apache|tomcat" | grep -v grep && \
sudo tail -50 /var/log/nginx/error.log
```

---

## 2. Nginx 점검

### 2.1 기본 정보

| 항목 | 경로 (Ubuntu/Debian) | 경로 (RHEL/CentOS) |
|------|--------------------|-------------------|
| 설정 파일 | `/etc/nginx/nginx.conf` | 동일 |
| 사이트 설정 | `/etc/nginx/sites-enabled/` | `/etc/nginx/conf.d/` |
| 액세스 로그 | `/var/log/nginx/access.log` | 동일 |
| 에러 로그 | `/var/log/nginx/error.log` | 동일 |

### 2.2 기본 점검 명령어

```bash
systemctl status nginx
sudo nginx -t               # 설정 문법 검증 (필수!)
sudo nginx -T | less        # 설정 확인 (풀 덤프)
nginx -V                    # 컴파일 옵션 / 설치 모듈
ps aux | grep nginx
sudo ss -tlnp | grep nginx
```

### 2.3 Nginx 제어

```bash
sudo systemctl reload nginx     # 설정 reload (무중단)
sudo nginx -s reload
sudo nginx -s reopen            # 로그 파일 재열기
sudo nginx -s quit              # graceful 종료
```

### 2.4 에러 로그 주요 패턴

```bash
sudo tail -f /var/log/nginx/error.log
sudo awk '{print $4}' /var/log/nginx/error.log | sort | uniq -c | sort -rn
```

| 에러 메시지 | 원인 | 조치 |
|-----------|------|------|
| `connect() failed (111: Connection refused)` | 업스트림 서버 다운 | 백엔드 서비스 확인 |
| `upstream timed out (110: Connection timed out)` | 업스트림 응답 느림 | proxy_read_timeout 조정 |
| `no live upstreams` | 모든 업스트림 서버 fail | 백엔드 전체 확인 |
| `SSL_do_handshake() failed` | SSL 핸드셰이크 실패 | 인증서, TLS 버전 확인 |
| `client intended to send too large body` | 업로드 크기 초과 | `client_max_body_size` 증가 |
| `Too many open files` | 파일 디스크립터 부족 | `worker_rlimit_nofile` 증가 |
| `worker_connections are not enough` | worker 연결 한계 | `worker_connections` 증가 |
| `upstream prematurely closed connection` | 백엔드가 먼저 끊음 | 백엔드 타임아웃 설정 확인 |
| `permission denied` | 권한 문제 | 파일/디렉토리 권한 및 SELinux 확인 |

### 2.5 액세스 로그 분석

```bash
sudo tail -f /var/log/nginx/access.log

# 상태코드별 집계
sudo awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c | sort -rn

# 많이 요청한 IP TOP 10
sudo awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head

# 5xx 에러 요청
sudo awk '$9 ~ /^5/' /var/log/nginx/access.log | tail -20

# 요청 URL TOP 10
sudo awk '{print $7}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head
```

### 2.6 커넥션 상태 확인 (stub_status)

`nginx.conf`에 활성화 필요:
```nginx
location /nginx_status {
    stub_status;
    allow 127.0.0.1;
    deny all;
}
```

```bash
curl http://localhost/nginx_status
```

| 항목 | 의미 |
|------|------|
| Active connections | 현재 열린 연결 수 |
| Reading | 요청 헤더 읽는 중 |
| Writing | 응답 쓰는 중 |
| Waiting | Keep-alive 대기 |

### 2.7 502 Bad Gateway 대응

```bash
sudo tail -50 /var/log/nginx/error.log
curl http://<백엔드IP>:<포트>/health
grep -r "proxy_pass\|upstream" /etc/nginx/
ss -tlnp | grep <백엔드포트>
# proxy_connect_timeout / proxy_read_timeout / proxy_send_timeout 확인
```

---

## 3. Apache HTTPD 점검

### 3.1 기본 정보

| 항목 | Ubuntu/Debian | RHEL/CentOS |
|------|--------------|-------------|
| 서비스명 | `apache2` | `httpd` |
| 설정 파일 | `/etc/apache2/apache2.conf` | `/etc/httpd/conf/httpd.conf` |
| 액세스 로그 | `/var/log/apache2/access.log` | `/var/log/httpd/access_log` |
| 에러 로그 | `/var/log/apache2/error.log` | `/var/log/httpd/error_log` |

### 3.2 기본 명령어

```bash
systemctl status apache2          # Ubuntu
systemctl status httpd            # RHEL
sudo apachectl -t                 # 설정 문법 검증
apachectl -S                      # 가상호스트 확인
apachectl -M                      # 로드된 모듈
sudo apachectl graceful           # 무중단 reload
```

### 3.3 Apache 주요 에러

| 에러 | 원인 | 조치 |
|------|------|------|
| `AH00558: Could not reliably determine the server's fully qualified domain name` | ServerName 미설정 | `ServerName localhost` 추가 |
| `server reached MaxRequestWorkers setting` | worker 한계 도달 | MaxRequestWorkers 증가 |
| `AH01797: client denied by server configuration` | Require 디렉티브 차단 | 허용 규칙 추가 |
| `PHP Fatal error: Allowed memory size exhausted` | PHP 메모리 한계 | php.ini의 memory_limit 증가 |

### 3.4 MPM 모드 확인

```bash
apache2ctl -V | grep -i mpm
```

| MPM | 특징 | 용도 |
|-----|------|------|
| prefork | 프로세스 기반 | 스레드 안전하지 않은 모듈 (mod_php 등) |
| worker | 프로세스 + 스레드 | 일반적 |
| event | worker 개선형, Keep-alive 효율적 | 최신 권장 |

---

## 4. Tomcat 점검

### 4.1 기본 정보

| 항목 | 경로 |
|------|------|
| 설치 디렉토리 | `$CATALINA_HOME` (예: `/opt/tomcat`) |
| 주요 설정 | `server.xml`, `web.xml`, `context.xml` |
| 주요 로그 | `catalina.out`, `localhost.<date>.log` |

### 4.2 기본 명령어

```bash
systemctl status tomcat
ps -ef | grep java
jps -l                              # Java 프로세스 목록
sudo ss -tlnp | grep -E ':(8080|8009|8005)'
tail -f $CATALINA_HOME/logs/catalina.out
grep -i "error\|exception\|severe" $CATALINA_HOME/logs/catalina.out | tail -50
```

### 4.3 Tomcat JVM 상태 확인

```bash
jstat -gcutil <PID> 1000 10         # GC 사용률 (1초마다 10회)
jmap -heap <PID>                    # 힙 사용량 상세
jmap -dump:live,format=b,file=/tmp/heap.hprof <PID>   # 힙 덤프

# 스레드 덤프 (데드락/행 분석) - 3회 떠서 비교
for i in 1 2 3; do
  jstack <PID> > /tmp/thread-$i.dump
  sleep 5
done

# 스레드 상태별 개수
awk '/java.lang.Thread.State/ {print $NF}' /tmp/thread-1.dump | sort | uniq -c
```

### 4.4 Tomcat 튜닝 점검 항목

**server.xml 주요 설정:**

```xml
<Connector port="8080" protocol="HTTP/1.1"
    maxThreads="200"
    minSpareThreads="25"
    acceptCount="100"
    connectionTimeout="20000"
    maxConnections="10000"
    enableLookups="false" />
```

**JVM 옵션 (setenv.sh):**

```bash
JAVA_OPTS="-server \
  -Xms2g -Xmx2g \
  -XX:+UseG1GC \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/var/log/tomcat/"
```

### 4.5 Tomcat 자주 보는 장애

#### OutOfMemoryError

```bash
grep -A 20 "OutOfMemoryError" $CATALINA_HOME/logs/catalina.out
# "Java heap space" → -Xmx 증가
# "Metaspace" → -XX:MaxMetaspaceSize 증가
# "GC overhead limit exceeded" → 메모리 누수 의심
# "unable to create new native thread" → OS 스레드 한계
```

#### 응답 없음 (Hang)

```bash
# 스레드 상태 분석
awk '/java.lang.Thread.State/ {print $NF}' /tmp/thread-1.dump | sort | uniq -c
# BLOCKED 많음 → 락 경합
# WAITING 많음 → 외부 호출 대기 (DB, API)
grep -B 2 -A 20 "BLOCKED" /tmp/thread-1.dump
```

#### 배포 실패

```bash
# work 디렉토리 클리어 후 재시작 (캐시 이슈)
sudo systemctl stop tomcat
sudo rm -rf $CATALINA_HOME/work/Catalina/
sudo systemctl start tomcat
```

---

## 5. HTTP 상태 코드별 대응

| 코드 | 의미 | 점검 사항 |
|------|------|---------|
| **400** | Bad Request | 요청 헤더/바디 문법 오류, 너무 긴 URL |
| **401** | Unauthorized | 인증 필요/실패 |
| **403** | Forbidden | 파일 권한, SELinux, 디렉토리 리스팅 |
| **404** | Not Found | 경로 오타, 배포 누락, rewrite 규칙 |
| **413** | Payload Too Large | `client_max_body_size` (Nginx), `LimitRequestBody` (Apache) |
| **499** | Client Closed Request (Nginx) | 클라이언트가 타임아웃 전에 끊음 |
| **500** | Internal Server Error | 어플리케이션 로그 확인 |
| **502** | Bad Gateway | 업스트림 서버 응답 실패 |
| **503** | Service Unavailable | 서버 과부하, worker 포화 |
| **504** | Gateway Timeout | 업스트림 응답 지연, 타임아웃 설정 확인 |

---

## 6. 실전 시나리오

### 시나리오 1: 502 Bad Gateway 발생

```bash
# 1. Nginx 로그에서 upstream 에러 확인
sudo tail -30 /var/log/nginx/error.log
# → "connect() failed (111: Connection refused)"

# 2. 업스트림 설정 확인
sudo grep -A 2 "upstream\|proxy_pass" /etc/nginx/conf.d/*.conf

# 3. 백엔드 생존 확인
curl http://127.0.0.1:8080/

# 4. 백엔드 로그 확인
tail -100 $CATALINA_HOME/logs/catalina.out

# 5. 재시작
sudo systemctl restart tomcat
```

### 시나리오 2: Tomcat 응답 없음 (Hang)

```bash
# 스레드 덤프 3회 (5초 간격)
for i in 1 2 3; do
  jstack <PID> > /tmp/thread-$i.dump
  sleep 5
done

# 상태별 집계
awk '/java.lang.Thread.State/ {print $NF}' /tmp/thread-1.dump | sort | uniq -c

# 원인별 조치
# BLOCKED 많음 → DB 커넥션 풀 고갈, 데드락
# WAITING 많음 → 외부 API 대기
```

### 시나리오 3: Tomcat OutOfMemory

```bash
grep -B 2 "OutOfMemoryError" $CATALINA_HOME/logs/catalina.out | tail -20

# 힙 사용률 모니터링 (지속 증가 = 누수)
jstat -gcutil <PID> 5000 20

# 힙 덤프 생성
jmap -dump:live,format=b,file=/tmp/heap.hprof <PID>
# MAT(Eclipse Memory Analyzer)로 Dominator Tree 분석
```

### 시나리오 4: Nginx 404 에러

```bash
# 설정 파일에서 location 블록 확인
sudo nginx -T | grep -A 5 "location"
sudo nginx -T | grep -E "root|alias"

# 실제 파일 존재 여부
ls -la /var/www/html/<경로>

# Nginx 실행 사용자 확인
ps aux | grep nginx | head -2

# SPA 라우팅 이슈
grep "try_files" /etc/nginx/conf.d/*.conf
```

### 시나리오 5: Apache 응답 느림

```bash
grep "server reached MaxRequestWorkers" /var/log/apache2/error.log
curl http://localhost/server-status?auto    # mod_status 활성화 시

# 슬로우 요청 (LogFormat에 %D 추가 시)
awk '$NF > 5000000' /var/log/apache2/access.log   # 5초 이상
```

---

## 7. 운영 체크리스트

### 일상 점검
- [ ] 서비스 프로세스 정상 기동 (`systemctl status`)
- [ ] 포트 LISTEN 정상 (`ss -tlnp`)
- [ ] 에러 로그 급증 없음 (`tail error.log`)
- [ ] 디스크 사용률 80% 미만 (`df -h`)

### 주기 점검
- [ ] JVM 힙 사용률 추이 (메모리 누수 감지)
- [ ] 응답 시간 추이
- [ ] 5xx 에러 발생 빈도
- [ ] SSL 인증서 만료일 (`openssl s_client -connect host:443 2>/dev/null | openssl x509 -noout -dates`)

### 배포 전후
- [ ] 설정 파일 문법 검증 (`nginx -t`, `apachectl -t`)
- [ ] graceful reload 가능 여부 확인
- [ ] 헬스체크 엔드포인트 정상 응답
