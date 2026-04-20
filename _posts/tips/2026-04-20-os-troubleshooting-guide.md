---
title: "Linux OS 트러블슈팅 실무 가이드"
date: 2026-04-20 02:00:00 +0900
categories: [Tips, Troubleshooting]
tags: [linux, os, troubleshooting, cpu, memory, disk, process, systemd, journalctl]
description: "Linux 서버에서 발생하는 다양한 장애를 OS 레벨에서 진단하는 실무 가이드 - CPU, 메모리, 디스크, 프로세스, 로그, 서비스 점검"
---

> Linux 서버에서 발생하는 다양한 장애를 OS 레벨에서 진단하는 실무 가이드입니다.

## 목차

1. [트러블슈팅 기본 원칙](#1-트러블슈팅-기본-원칙)
2. [CPU 점검](#2-cpu-점검)
3. [메모리 점검](#3-메모리-점검)
4. [디스크 / 파일시스템 점검](#4-디스크--파일시스템-점검)
5. [디스크 I/O 점검](#5-디스크-io-점검)
6. [프로세스 점검](#6-프로세스-점검)
7. [시스템 로그 점검](#7-시스템-로그-점검)
8. [서비스 / systemd 점검](#8-서비스--systemd-점검)
9. [사용자 / 권한 점검](#9-사용자--권한-점검)
10. [부팅 / 커널 점검](#10-부팅--커널-점검)
11. [보안 / 무결성 점검](#11-보안--무결성-점검)
12. [실전 트러블슈팅 시나리오](#12-실전-트러블슈팅-시나리오)

---

## 1. 트러블슈팅 기본 원칙

### 1.1 진단 순서

서버 이상 시 아래 순서로 빠르게 전체 상태를 훑습니다.

```
1. 전체 부하 확인       → top, htop, uptime
2. 자원 사용량 확인     → free, df, iostat
3. 의심 자원 심층 분석   → vmstat, sar, pidstat
4. 프로세스 식별        → ps, lsof
5. 로그 확인            → journalctl, /var/log/*
6. 서비스 상태          → systemctl
```

### 1.2 자주 쓰는 1차 점검 명령어 5종 세트

서버 들어가서 일단 칠 명령어:

```bash
uptime              # 부하 평균
free -h             # 메모리
df -h               # 디스크
top                 # 종합 (q로 종료)
journalctl -xe      # 최근 에러 로그
```

### 1.3 도구 분류

| 카테고리 | 실시간 모니터링 | 통계/스냅샷 |
|---------|---------------|------------|
| 종합 | `top`, `htop` | `uptime`, `vmstat` |
| CPU | `mpstat 1` | `sar -u` |
| 메모리 | `free -s 1` | `vmstat -s` |
| 디스크 I/O | `iotop`, `iostat 1` | `sar -d` |
| 네트워크 | `iftop`, `nethogs` | `sar -n` |
| 프로세스 | `pidstat 1` | `ps aux` |

---

## 2. CPU 점검

### 2.1 "CPU가 너무 높아요"

```bash
# 1. 전체 부하 확인 (Load Average)
uptime
# load average: 1.5, 2.0, 1.8
# (1분, 5분, 15분 평균) → CPU 코어 수 초과 시 부하 발생

# CPU 코어 수 확인
nproc
cat /proc/cpuinfo | grep processor | wc -l

# 2. 실시간 CPU 사용 프로세스 확인
top              # P키: CPU 정렬
htop             # 더 보기 편함 (별도 설치)

# CPU 많이 쓰는 프로세스 TOP 10
ps aux --sort=-%cpu | head -11

# 3. 코어별 상세 사용률
mpstat -P ALL 1 5    # 1초 간격 5회

# 4. 특정 프로세스 CPU 분석
pidstat -p <PID> 1
top -H -p <PID>      # 스레드별 CPU
```

### 2.2 CPU 항목 의미

`top` 출력에서 CPU 라인:

```
%Cpu(s):  5.2 us,  2.1 sy,  0.0 ni, 92.5 id,  0.1 wa,  0.0 hi,  0.1 si,  0.0 st
```

| 항목 | 의미 | 높을 때 의심 |
|------|------|------------|
| `us` (user) | 사용자 프로세스 | 어플리케이션 부하 |
| `sy` (system) | 커널 프로세스 | 시스템 콜 과다, 컨텍스트 스위칭 |
| `ni` (nice) | 우선순위 낮은 프로세스 | 보통 무시 |
| `id` (idle) | 유휴 | 높을수록 좋음 |
| `wa` (iowait) | I/O 대기 | **디스크 병목** |
| `hi` (hardware irq) | 하드웨어 인터럽트 | 네트워크/디스크 폭주 |
| `si` (software irq) | 소프트웨어 인터럽트 | 네트워크 폭주 |
| `st` (steal) | 가상화 환경 다른 VM에 뺏김 | **클라우드 노이지 네이버** |

### 2.3 Load Average 해석

```bash
uptime
# 14:23:01 up 30 days,  load average: 4.5, 3.2, 2.1
```

- **Load = 실행 중 + 실행 대기 중 프로세스 수**
- CPU 코어 수가 4개라면:
  - Load 4.0 = 100% (포화)
  - Load 4.5 = 과부하 (대기 중)
  - Load 1.0 = 25%

```bash
# 코어 수 대비 load 계산
echo "scale=2; $(uptime | awk -F'load average:' '{print $2}' | cut -d, -f1) / $(nproc)"
```

### 2.4 컨텍스트 스위칭 / 인터럽트

```bash
vmstat 1 5
# r: 실행 대기 프로세스 수
# cs: 컨텍스트 스위칭/초
# in: 인터럽트/초
```

`cs`가 비정상적으로 높으면(수십만/초) 스레드 과다 또는 락 경합 의심.

---

## 3. 메모리 점검

### 3.1 "메모리가 부족해요"

```bash
# 1. 전체 메모리 상태
free -h

# 2. 메모리 많이 쓰는 프로세스 TOP 10
ps aux --sort=-%mem | head -11

# 3. 실시간
top              # M키: 메모리 정렬

# 4. 상세 정보
cat /proc/meminfo

# 5. 슬랩 메모리 (커널 메모리)
slabtop
```

### 3.2 free 출력 해석

```
              total        used        free      shared  buff/cache   available
Mem:           7.7Gi       2.1Gi       3.2Gi       100Mi       2.4Gi       5.3Gi
```

| 항목 | 의미 |
|------|------|
| `total` | 총 메모리 |
| `used` | 사용 중 |
| `free` | 완전 미사용 (보통 작음, 정상) |
| `buff/cache` | 캐시 메모리 (필요 시 즉시 반환됨) |
| **`available`** | **실제 사용 가능 (이 값을 보세요)** |

**`free`가 작아도 `available`이 충분하면 정상**입니다. Linux는 빈 메모리를 캐시로 활용합니다.

### 3.3 OOM (Out Of Memory) 확인

```bash
# OOM Killer가 발동했는지 확인
dmesg | grep -i "killed process\|out of memory\|oom"
journalctl -k | grep -i "oom\|killed process"

# OOM 점수 (높을수록 OOM 시 먼저 죽음)
cat /proc/<PID>/oom_score
```

### 3.4 Swap 사용 확인

```bash
free -h
swapon -s

# 어떤 프로세스가 swap을 쓰는지
for pid in $(ls /proc/ | grep -E '^[0-9]+$'); do
  swap=$(grep VmSwap /proc/$pid/status 2>/dev/null | awk '{print $2}')
  if [ -n "$swap" ] && [ "$swap" -gt 0 ]; then
    name=$(cat /proc/$pid/comm 2>/dev/null)
    echo "PID:$pid Swap:${swap}KB Name:$name"
  fi
done | sort -k2 -t: -nr | head -10
```

### 3.5 메모리 누수 의심 시

```bash
# 시간에 따른 메모리 변화 모니터링
while true; do
  date
  ps -p <PID> -o pid,vsz,rss,cmd
  sleep 10
done

# 또는
pidstat -r -p <PID> 5
```

---

## 4. 디스크 / 파일시스템 점검

### 4.1 "디스크가 가득 찼어요"

```bash
# 1. 전체 디스크 사용량
df -h

# 2. 사용량 많은 디렉토리 찾기
sudo du -h --max-depth=1 / 2>/dev/null | sort -hr | head -20
sudo du -h --max-depth=1 /var 2>/dev/null | sort -hr | head -20

# 3. 큰 파일 찾기 (TOP 20)
sudo find / -type f -size +100M 2>/dev/null | xargs ls -lh 2>/dev/null | sort -k5 -hr | head -20

# 4. 최근에 커진 파일 찾기 (24시간 내)
sudo find / -type f -mtime -1 -size +50M 2>/dev/null

# 5. inode 사용량 (파일 개수 한계)
df -i
# IUse%가 100%면 파일 개수 한계 (디스크 용량은 남아도 파일 못 만듦)
```

### 4.2 디스크 가득 찼을 때 확인 우선순위

```bash
# 1. 로그 디렉토리 (가장 흔한 원인)
sudo du -sh /var/log/
sudo du -sh /var/log/* | sort -hr | head -10

# 2. journalctl 로그 (systemd)
sudo journalctl --disk-usage

# 3. Docker (있는 경우)
sudo du -sh /var/lib/docker/
docker system df
docker system prune -a    # 정리 (주의)

# 4. 패키지 캐시
sudo apt-get clean        # Ubuntu/Debian
sudo yum clean all        # RHEL/CentOS

# 5. 사용자 홈 디렉토리
sudo du -sh /home/* | sort -hr | head -10

# 6. /tmp 임시 파일
sudo du -sh /tmp/
```

### 4.3 삭제했는데 용량이 안 줄어요

프로세스가 파일을 잡고 있으면 삭제해도 디스크가 반환되지 않습니다.

```bash
# 삭제됐지만 열려있는 파일 찾기
sudo lsof | grep deleted | head -20

# 해결: 해당 프로세스 재시작 또는 종료
# 또는 프로세스가 쓰는 파일 비우기
> /proc/<PID>/fd/<fd번호>
```

### 4.4 마운트 / 파일시스템 확인

```bash
mount | column -t
findmnt
df -T                           # 파일시스템 타입
cat /etc/fstab
sudo dmesg | grep -i "ext4\|xfs\|filesystem\|error"
sudo fsck /dev/sda1             # 마운트 해제 후 사용
```

---

## 5. 디스크 I/O 점검

### 5.1 "디스크가 느려요"

```bash
# 전체 I/O 부하 확인
iostat -x 1 5
# %util: 디스크 사용률 (90% 이상 포화)
# await: I/O 평균 대기시간 (ms)

# iowait 확인 (CPU가 디스크 기다리는 시간)
top                 # wa 항목
vmstat 1            # wa 칼럼

# 어떤 프로세스가 I/O를 일으키는지
sudo iotop -o

# 프로세스별 I/O
pidstat -d 1
```

### 5.2 iostat 해석

| 항목 | 의미 | 위험 임계 |
|------|------|---------|
| `r/s`, `w/s` | 초당 read/write 횟수 (IOPS) | 디스크 사양 대비 |
| `rkB/s`, `wkB/s` | 초당 read/write 데이터량 | 대역폭 한계 |
| `await` | I/O 평균 대기시간(ms) | 10ms 이상 (HDD), 1ms 이상 (SSD) |
| `%util` | 디스크 사용률 | 80% 이상 시 병목 의심 |

### 5.3 디스크 성능 테스트

```bash
# 쓰기 속도 테스트
dd if=/dev/zero of=/tmp/test.img bs=1M count=1024 oflag=direct
rm /tmp/test.img

# 읽기 속도 테스트
hdparm -t /dev/sda
```

---

## 6. 프로세스 점검

### 6.1 프로세스 확인

```bash
ps aux | grep <프로세스명>
pgrep -a <프로세스명>
ps -p <PID> -o pid,ppid,user,cmd,stat,start,etime
pstree -p
```

### 6.2 프로세스 STAT 코드

| 코드 | 의미 |
|------|------|
| `R` | Running (실행 중/실행 가능) |
| `S` | Sleeping (인터럽트 가능) |
| `D` | Uninterruptible Sleep (보통 I/O 대기, 죽일 수 없음) |
| `T` | Stopped (정지됨) |
| `Z` | Zombie (좀비, 부모가 회수 안 함) |

### 6.3 프로세스 강제 종료

```bash
kill <PID>          # 정상 종료 (TERM)
kill -9 <PID>       # 강제 종료 (KILL, DB 등 데이터 손상 위험)
pkill -f <프로세스명>
```

**주의:** `D` 상태(Uninterruptible)는 `kill -9`로도 못 죽임 (커널이 잡고 있음)

### 6.4 프로세스가 사용 중인 파일 / 포트

```bash
sudo lsof -p <PID>              # 프로세스가 연 파일/소켓 전부
sudo lsof -i :443               # 특정 포트 사용 프로세스
sudo lsof /var/log/app.log      # 특정 파일 사용 프로세스
sudo lsof +D /mnt/data          # 특정 디렉토리 사용 프로세스
```

### 6.5 프로세스 상세 추적

```bash
sudo strace -p <PID>            # 시스템 콜 추적
cat /proc/<PID>/environ | tr '\0' '\n'   # 환경변수
ls -la /proc/<PID>/cwd          # 작업 디렉토리
ls -la /proc/<PID>/exe          # 실행 파일
```

---

## 7. 시스템 로그 점검

### 7.1 어떤 로그를 볼까

```
/var/log/
├── syslog            # Ubuntu/Debian 시스템 로그
├── messages          # RHEL/CentOS 시스템 로그
├── auth.log          # 인증 관련 (Ubuntu)
├── secure            # 인증 관련 (RHEL)
├── kern.log          # 커널 로그
└── dmesg             # 부팅 시 커널 로그
```

### 7.2 systemd 시스템 (journalctl)

```bash
journalctl -b                           # 최근 부팅 이후
journalctl -b -1                        # 이전 부팅 로그
journalctl -f                           # 실시간 follow
journalctl -p err                       # 에러만
journalctl -u nginx -f                  # 특정 서비스
journalctl --since "2 hours ago"        # 시간 범위
journalctl -k                           # 커널 로그만
journalctl --disk-usage                 # 디스크 사용량
sudo journalctl --vacuum-time=7d        # 7일 초과 삭제
```

### 7.3 dmesg (커널 로그)

```bash
dmesg -T                    # 사람이 읽을 수 있는 시간
dmesg -w                    # 실시간
dmesg --level=err,crit      # 에러만
dmesg | grep -i "oom\|killed"
dmesg | grep -i "i/o\|disk\|sda"
```

---

## 8. 서비스 / systemd 점검

### 8.1 서비스 상태 확인

```bash
systemctl status nginx
systemctl --failed                          # 실패한 서비스
systemctl list-unit-files --state=enabled   # 자동 시작 서비스
```

### 8.2 서비스 제어

```bash
sudo systemctl start|stop|restart nginx
sudo systemctl reload nginx                 # 설정 reload (무중단)
sudo systemctl enable --now nginx           # 즉시 시작 + 자동 시작
```

### 8.3 서비스 안 떠요

```bash
systemctl status nginx -l
journalctl -u nginx -n 50
nginx -t                    # 설정 파일 검증
sudo ss -tlnp | grep :80    # 포트 충돌 확인
sudo getenforce             # SELinux 확인
```

### 8.4 부팅 분석

```bash
systemd-analyze                     # 부팅 시간
systemd-analyze blame | head -20    # 가장 오래 걸린 서비스
systemd-analyze critical-chain      # 크리티컬 패스
```

---

## 9. 사용자 / 권한 점검

```bash
id && who && w          # 현재 사용자, 로그인 현황
last -n 20              # 최근 로그인
sudo lastb              # 실패한 로그인 시도

# SSH 키 파일 권한 (자주 발생하는 문제)
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

## 10. 부팅 / 커널 점검

```bash
uname -r                            # 커널 버전
cat /proc/cmdline                   # 부팅 커널 매개변수
lsmod                               # 커널 모듈 목록

# sysctl (커널 파라미터)
sysctl net.ipv4.ip_forward
sudo sysctl -w net.ipv4.ip_forward=1    # 임시 변경
sudo vi /etc/sysctl.d/99-custom.conf    # 영구 변경
sudo sysctl -p

# 재부팅 기록
last reboot
last -x | head
```

---

## 11. 보안 / 무결성 점검

```bash
# 의심 활동
sudo grep "Failed password" /var/log/auth.log | tail -20
sudo grep "useradd\|new user" /var/log/auth.log
ss -tan state established

# 의심스러운 cron job
sudo crontab -l
ls /etc/cron.d/ /etc/cron.daily/

# SUID 비트 있는 파일 (권한 상승 가능)
sudo find / -perm -4000 -type f 2>/dev/null
```

---

## 12. 실전 트러블슈팅 시나리오

### 시나리오 1: "서버가 느려요"

```bash
uptime                          # 부하 평균 확인
top                             # CPU vs I/O vs 메모리 식별
# %Cpu wa 높음 → I/O 병목: iostat -x 1, sudo iotop
# us 높음 → CPU 병목: ps aux --sort=-%cpu | head
# 메모리 swap 사용 → 메모리 부족: free -h
journalctl -p err --since "10 minutes ago"
```

### 시나리오 2: "디스크 가득 찼다고 알람"

```bash
df -h && df -i
sudo du -h --max-depth=1 / 2>/dev/null | sort -hr | head
sudo find / -type f -size +500M 2>/dev/null | xargs ls -lh 2>/dev/null | sort -k5 -hr
sudo journalctl --disk-usage
sudo lsof | grep deleted        # 삭제했는데 안 줄면 확인
```

### 시나리오 3: "프로세스가 죽었어요"

```bash
journalctl -u <서비스명> --since "1 hour ago"
dmesg | grep -i "killed process\|oom"   # OOM Killer 의심
coredumpctl list                         # 코어 덤프 확인
systemctl list-dependencies <서비스명>   # 의존 서비스 실패 여부
```

### 시나리오 4: "재부팅 후 이상해요"

```bash
journalctl -b -p err            # 부팅 중 에러
systemctl --failed              # 실패한 서비스
systemd-analyze blame | head    # 부팅 시간 분석
journalctl -b -1 -p err         # 이전 부팅과 비교
```

### 시나리오 5: "서비스가 시작 안 돼요"

```bash
systemctl status <서비스> -l
journalctl -u <서비스> -n 50
nginx -t                        # 설정 파일 검증
sudo ss -tlnp | grep :<포트>    # 포트 충돌
df -h && free -h                # 디스크/메모리 부족 아닌지
```

### 시나리오 6: "SSH 접속이 안 돼요"

```bash
systemctl status sshd
sudo sshd -t                            # 설정 검증
sudo ss -tlnp | grep ssh
sudo ufw status                         # 방화벽 (Ubuntu)
sudo fail2ban-client status sshd        # fail2ban 차단 여부
```

---

## 13. 체크리스트 (서버 받은 후 30초 점검)

```bash
uptime && free -h && df -h && systemctl --failed && journalctl -p err -b --no-pager | tail -20
```

- [ ] Load Average가 코어 수보다 작은가?
- [ ] 메모리 available이 충분한가?
- [ ] 디스크 사용률이 80% 미만인가?
- [ ] 실패한 서비스가 없는가?
- [ ] 최근 부팅 이후 에러 로그가 없는가?

---

## 14. 배포판별 차이

| 항목 | Ubuntu/Debian | RHEL/CentOS |
|------|--------------|-------------|
| 패키지 관리자 | `apt` | `yum`/`dnf` |
| 시스템 로그 | `/var/log/syslog` | `/var/log/messages` |
| 인증 로그 | `/var/log/auth.log` | `/var/log/secure` |
| 방화벽 | `ufw`, `iptables` | `firewalld`, `iptables` |
| SELinux | 보통 비활성 | 기본 활성 |
| 패키지 무결성 | `debsums` | `rpm -Va` |

---

## 15. 운영 환경 주의사항

- **`kill -9` 남발 금지** — DB 등은 데이터 손상 위험. SIGTERM부터 시도.
- **`rm -rf` 신중히** — 변수 비어있으면 `/` 삭제
- **iotop, strace는 부하 발생** — 운영 중 단발성으로만 사용
- **로그 삭제 신중히** — 감사/법적 요구 있을 수 있음. truncate(`>`)로 비우는 게 안전
- **fsck는 마운트 해제 후** — 마운트된 상태에서 실행 시 파일시스템 손상
