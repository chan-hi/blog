---
title: "애플리케이션 트러블슈팅 실무 가이드 (Java / Node.js / Python)"
date: 2026-04-20 05:00:00 +0900
categories: [Tips, Troubleshooting]
tags: [java, spring, nodejs, python, troubleshooting, oom, memory-leak, jvm, pm2]
description: "Java(Spring), Node.js, Python 애플리케이션 장애 진단 실무 가이드 - OOM, 메모리 누수, Hang 분석, APM"
---

> Java(Spring), Node.js, Python 등 주요 애플리케이션 장애 진단 실무 가이드입니다.

## 목차

1. [공통 점검 흐름](#1-공통-점검-흐름)
2. [Java / Spring 애플리케이션](#2-java--spring-애플리케이션)
3. [Node.js 애플리케이션](#3-nodejs-애플리케이션)
4. [Python 애플리케이션](#4-python-애플리케이션)
5. [공통 장애 패턴 분석](#5-공통-장애-패턴-분석)
6. [실전 시나리오](#6-실전-시나리오)

---

## 1. 공통 점검 흐름

### 1.1 애플리케이션 장애 표준 대응 순서

```
1. 증상 재현            → curl, 브라우저 (어떤 요청에서?)
2. 프로세스 생존 확인    → ps, systemctl
3. 포트 LISTEN 확인     → ss -tlnp
4. 헬스체크 응답         → /health, /actuator/health
5. 애플리케이션 로그     → 에러 메시지, 스택트레이스
6. 리소스 사용량         → CPU, 메모리, 스레드 수
7. 의존성 점검          → DB, 캐시, 외부 API
8. 심층 분석            → 스레드 덤프, 힙 덤프, 프로파일링
```

### 1.2 어플리케이션 장애 분류

| 분류 | 증상 | 주 원인 |
|------|------|--------|
| 응답 없음 (Hang) | 요청이 응답 없이 대기 | 데드락, 외부 호출 대기 |
| 응답 느림 | 정상이지만 느림 | DB 쿼리, GC, 외부 API |
| 5xx 에러 | 내부 서버 에러 | 코드 버그, 의존성 실패 |
| 다운 | 프로세스 죽음 | OOM, 크래시 |
| 자원 누수 | 시간 지나면 느려짐/죽음 | 메모리/커넥션 누수 |

---

## 2. Java / Spring 애플리케이션

### 2.1 기본 점검 명령어

```bash
ps -ef | grep java
jps -l                              # Java 프로세스 목록
jps -lvm                            # JVM 옵션 포함
java -version
jcmd <PID> VM.command_line          # JVM 시작 옵션
```

### 2.2 JVM 모니터링 도구

#### `jstat` - GC / JVM 통계

```bash
jstat -gcutil <PID> 1000 10         # GC 사용률 (1초 간격, 10회)
jstat -gc <PID> 1000 10             # GC 상세 통계
```

**해석:**
- `FGC` (Full GC) 빈번 → 메모리 부족, 누수
- `FGCT/FGC` (평균 Full GC 시간) > 1초 → STW(Stop-The-World) 길어 응답성 저하
- `OU` (Old Gen) 지속 증가 → 메모리 누수 가능성

#### `jmap` - 힙 분석

```bash
jmap -heap <PID>                                          # 힙 요약
jmap -histo:live <PID> | head -30                        # 살아있는 객체 통계
jmap -dump:live,format=b,file=/tmp/heap.hprof <PID>      # 힙 덤프
```

#### `jstack` - 스레드 덤프

```bash
# 3회 떠서 비교 (5초 간격)
for i in 1 2 3; do
  jstack -l <PID> > /tmp/thread-$i.dump
  sleep 5
done

# 스레드 상태별 개수
awk '/java.lang.Thread.State/ {print $NF}' /tmp/thread-1.dump | sort | uniq -c

# BLOCKED 스레드
grep -B 2 -A 20 "BLOCKED" /tmp/thread-1.dump

# CPU 많이 쓰는 스레드 찾기
top -H -p <PID>
# nid 확인 → printf "%x\n" <nid> → jstack에서 nid=0x<hex> 검색
```

#### `jcmd` - 통합 진단 도구 (권장)

```bash
jcmd <PID> Thread.print              # 스레드 덤프
jcmd <PID> GC.heap_info              # 힙 정보
jcmd <PID> GC.heap_dump /tmp/heap.hprof    # 힙 덤프
jcmd <PID> GC.class_histogram        # 클래스 통계
```

### 2.3 Spring Actuator 활용

```bash
curl http://localhost:8080/actuator/health | jq
curl http://localhost:8080/actuator/metrics/jvm.memory.used
curl http://localhost:8080/actuator/metrics/hikaricp.connections.active
curl http://localhost:8080/actuator/metrics/tomcat.threads.busy

# 로그 레벨 런타임 변경
curl -X POST http://localhost:8080/actuator/loggers/com.example \
     -H "Content-Type: application/json" \
     -d '{"configuredLevel": "DEBUG"}'

# 스레드 덤프 / 힙 덤프
curl http://localhost:8080/actuator/threaddump
curl http://localhost:8080/actuator/heapdump > heap.hprof
```

### 2.4 자주 보는 Java 에러

| 에러 | 원인 | 조치 |
|------|------|------|
| `OutOfMemoryError: Java heap space` | 힙 부족 또는 누수 | `-Xmx` 증가, 힙 덤프 분석 |
| `OutOfMemoryError: Metaspace` | 메타스페이스 부족 | `-XX:MaxMetaspaceSize` 증가 |
| `OutOfMemoryError: GC overhead limit exceeded` | GC 98% 차지 | 메모리 누수 의심, 힙 덤프 분석 |
| `OutOfMemoryError: unable to create new native thread` | OS 스레드 한계 | `ulimit -u` 확인, 스레드 누수 점검 |
| `StackOverflowError` | 무한 재귀 | 코드 점검, `-Xss` 증가 |
| `IllegalStateException: Connection pool exhausted` | DB 커넥션 풀 고갈 | 풀 크기, 누수 점검 |

### 2.5 메모리 누수 분석

```bash
# 1. 힙 사용률 추이 (지속 증가 = 누수 의심)
jstat -gcutil <PID> 5000 60

# 2. 힙 덤프
jmap -dump:live,format=b,file=/tmp/heap.hprof <PID>

# 3. MAT(Eclipse Memory Analyzer)로 분석
# - Leak Suspects 리포트
# - Dominator Tree
# - Histogram → 의심 클래스 → Path to GC Roots

# 4. JFR (JDK 11+)
jcmd <PID> JFR.start name=app duration=60s filename=/tmp/recording.jfr
```

---

## 3. Node.js 애플리케이션

### 3.1 기본 점검 명령어

```bash
ps -ef | grep node
sudo ss -tlnp | grep node
node -v && npm -v
```

### 3.2 PM2 사용 시

```bash
pm2 list                            # 프로세스 목록
pm2 show <app-name>                 # 상세 정보
pm2 logs <app-name> --err           # 에러 로그
pm2 restart <app-name>
pm2 reload <app-name>               # 무중단 (cluster 모드)
pm2 monit                           # 실시간 모니터링

# 메모리 한계 시 자동 재시작
pm2 start app.js --max-memory-restart 1G

# 클러스터 모드
pm2 start app.js -i max             # CPU 코어 수만큼
```

### 3.3 Node.js 디버깅

```bash
# Inspector 활성화
node --inspect=0.0.0.0:9229 app.js
# Chrome DevTools: chrome://inspect

# 실행 중인 프로세스에 inspector 활성화
kill -SIGUSR1 <PID>
```

### 3.4 메모리 누수

```bash
# 힙 한계 증가 (임시 조치)
node --max-old-space-size=4096 app.js

# 시간에 따른 메모리 추이
while true; do
  date
  ps -p <PID> -o rss
  sleep 10
done

# clinic.js (전문 도구)
npm install -g clinic
clinic doctor -- node app.js
clinic heapprofiler -- node app.js
```

### 3.5 Node.js 자주 보는 에러

| 에러 | 원인 | 조치 |
|------|------|------|
| `EADDRINUSE` | 포트 사용 중 | 다른 프로세스 종료 또는 포트 변경 |
| `ECONNREFUSED` | 연결 거부 | 대상 서비스 다운 |
| `EMFILE: too many open files` | 파일 디스크립터 한계 | `ulimit -n` 증가 |
| `JavaScript heap out of memory` | V8 힙 한계 (기본 1.5GB) | `--max-old-space-size=4096` |
| `UnhandledPromiseRejectionWarning` | Promise reject 미처리 | `.catch()` 추가 |
| `Cannot find module` | 모듈 누락 | `npm install` |

---

## 4. Python 애플리케이션

### 4.1 기본 점검

```bash
ps -ef | grep python
python3 --version
pip list
which python && echo $VIRTUAL_ENV   # 가상환경 확인
```

### 4.2 WSGI 서버 (Gunicorn / uWSGI)

```bash
# Gunicorn
ps -ef | grep gunicorn
tail -f /var/log/gunicorn/error.log
kill -HUP <master_pid>              # graceful reload

# uWSGI
uwsgi --reload /tmp/uwsgi.pid
```

### 4.3 py-spy (운영 환경 안전, 권장)

```bash
pip install py-spy

sudo py-spy top --pid <PID>          # 실시간 top
sudo py-spy dump --pid <PID>         # 스택 덤프 (행 분석)
sudo py-spy record -o profile.svg --pid <PID> --duration 30  # 프로파일링
```

### 4.4 Python 자주 보는 에러

| 에러 | 원인 | 조치 |
|------|------|------|
| `ModuleNotFoundError` | 모듈 없음 | `pip install`, 가상환경 활성화 |
| `MemoryError` | 메모리 부족 | 데이터 처리 방식 변경 (스트리밍) |
| `gunicorn: WORKER TIMEOUT` | worker 타임아웃 | `--timeout` 증가 또는 코드 최적화 |
| `OSError: [Errno 24] Too many open files` | 파일 디스크립터 한계 | `ulimit -n` 증가 |

---

## 5. 공통 장애 패턴 분석

### 5.1 응답 없음 (Hang) 분석 절차

```
1. 프로세스 살아있는가?
   - ps aux | grep <app>

2. CPU 사용 중인가?
   - 0% → 외부 대기 (DB, API)
   - 100% → 무한루프, GC 폭주

3. 어디서 멈췄나?
   - Java: jstack <PID>
   - Python: py-spy dump --pid <PID>
   - Node.js: kill -SIGUSR1 <PID>

4. 외부 의존성 점검
   - DB 응답 시간, Redis, 외부 API
```

### 5.2 메모리 누수 분석 절차

```
1. 시간에 따른 메모리 추이 확인
   - while sleep 60; do ps -p <PID> -o rss; done

2. 힙 덤프 (정상 시점, 누수 의심 시점)
   - Java: jmap -dump
   - Node.js: v8.writeHeapSnapshot
   - Python: tracemalloc

3. 두 덤프 비교 (증가하는 객체 식별)
   - Java: MAT
   - Node.js: Chrome DevTools Memory
   - Python: tracemalloc.compare_to()

4. 누수 원인 패턴
   - 정적 컬렉션에 객체 누적
   - 이벤트 리스너 미해제
   - 캐시 무한 증가
```

### 5.3 응답 지연 분석 절차

```
1. 어느 구간이 느린가? (분리)
   - 네트워크: ping, mtr
   - DB: 슬로우 쿼리 로그
   - 외부 API: 호출 시간 로깅

2. 패턴 확인
   - 항상 느림 → 코드 비효율, N+1 쿼리
   - 가끔 느림 → GC, 외부 API, 락 경합
   - 점점 느려짐 → 메모리 누수, 캐시 오염
   - 특정 시간만 → 배치 작업, 트래픽 급증
```

### 5.4 외부 의존성 점검

대부분의 어플리케이션 장애는 **자기 자신이 아닌 의존성**이 원인입니다.

```bash
mysql -h <DB호스트> -u <user> -p -e "SELECT 1"
redis-cli -h <Redis호스트> ping
curl -v -w "%{time_total}s\n" -o /dev/null https://external-api.com/health
dig external-api.com | grep "Query time"
mtr -r -c 50 external-api.com
```

---

## 6. 실전 시나리오

### 시나리오 1: Spring Boot 앱이 갑자기 응답 안 함

```bash
ps -ef | grep java          # 살아있음 확인
top -p <PID>                # CPU 0% → 어디선가 대기 중

# 스레드 덤프 3회
for i in 1 2 3; do
  jstack <PID> > /tmp/thread-$i.dump
  sleep 5
done

awk '/java.lang.Thread.State/ {print $NF}' /tmp/thread-1.dump | sort | uniq -c
# → 모든 스레드 WAITING → DB 커넥션 풀 고갈 의심

# HikariCP 상태
curl http://localhost:8080/actuator/metrics/hikaricp.connections.active
# active=20, max=20 → 고갈 확인

# DB 측 슬로우 쿼리 확인 → KILL 후 커넥션 풀 크기 증가
```

### 시나리오 2: Node.js 앱이 24시간 후 다운

```bash
pm2 logs <app-name> --err
grep -i "memory\|killed\|exit" ~/.pm2/logs/*-out.log
sudo dmesg | grep -i "killed process node"   # OOM Killer 확인

# 임시 조치: 메모리 한계 시 자동 재시작
pm2 start app.js --max-memory-restart 1G

# 근본 해결: 힙 스냅샷에서 누수 객체 식별
```

### 시나리오 3: Python (Django) 앱 응답 느림

```bash
ps -ef | grep gunicorn
sudo py-spy top --pid <worker_pid>      # 어떤 함수에 시간 쓰는지

# N+1 쿼리 패턴 확인 후 select_related, prefetch_related 활용
# Redis 캐시 적용 검토
```

### 시나리오 4: Java OOM 발생

```bash
grep -B 5 "OutOfMemoryError" /var/log/app/app.log
# "Java heap space" → 힙 부족
# "GC overhead limit exceeded" → 누수 가능성 높음

jstat -gcutil <PID> 5000
# OU(Old)가 100% + Full GC 후에도 안 줄어들면 누수

jmap -dump:live,format=b,file=/tmp/heap.hprof <PID>
# MAT으로 Leak Suspects 분석
```

### 시나리오 5: 특정 API만 5xx 발생

```bash
tail -f /var/log/app/app.log | grep -i "error\|exception"
# 스택트레이스에서 어떤 클래스/함수인지 확인
# DB 쿼리 실패? Redis 연결 실패? 외부 API 실패?

# 해당 시간대 트래픽 확인
awk '{print $4}' /var/log/nginx/access.log | sort | uniq -c
```

---

## 7. APM (Application Performance Monitoring) 도구

운영 환경에서는 APM 필수입니다.

| APM 도구 | 특징 | 적합 |
|---------|------|------|
| Pinpoint | 오픈소스, 한국 제작, Java 강점 | Java/Spring 환경 |
| Scouter | 오픈소스, 한국 제작 | Java |
| Datadog | SaaS, 다언어 지원 | 다양한 환경 |
| New Relic | SaaS, 강력한 분석 | 다양한 환경 |
| Jaeger / Zipkin | 분산 추적 | MSA |

---

## 8. 운영 체크리스트

### 일상 점검
- [ ] 프로세스 정상 기동
- [ ] 헬스체크 응답 (200 OK)
- [ ] 에러 로그 급증 없음
- [ ] CPU/메모리 정상 범위
- [ ] 5xx 에러율 < 0.1%

### 주기 점검
- [ ] 메모리 사용 추이 (누수 감지)
- [ ] GC 빈도/시간 (Java)
- [ ] 커넥션 풀 사용률
- [ ] 슬로우 쿼리 추이

---

## 9. 운영 시 주의사항

- **운영 환경에서 디버그 모드 금지** — 보안 위험, 성능 저하
- **로그 레벨 DEBUG는 일시적으로만** — I/O 부하
- **Heap dump는 정지/지연 발생** — 운영 중 신중히 (특히 큰 힙)
- **`jmap -histo:live`는 Full GC 트리거** — STW 발생
- **`py-spy`는 안전** — 대상 프로세스 정지 안 함
- **OOM 발생 시 힙 덤프 자동 생성 옵션 켜두기** (`-XX:+HeapDumpOnOutOfMemoryError`)
- **Inspector/Debugger 포트는 외부 노출 금지**
