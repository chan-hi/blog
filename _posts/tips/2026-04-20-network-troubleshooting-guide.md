---
title: "네트워크 트러블슈팅 실무 가이드"
date: 2026-04-20 01:00:00 +0900
categories: [Tips, Troubleshooting]
tags: [network, troubleshooting, curl, tcpdump, traceroute, mtr, nc, dns, linux]
description: "실제 장애 상황에서 어떤 명령어를 어떤 순서로 써야 하는지에 대한 실무 가이드"
---

> 실제 장애 상황에서 어떤 명령어를 어떤 순서로 써야 하는지에 대한 실무 가이드입니다.

## 목차

1. [트러블슈팅 기본 원칙](#1-트러블슈팅-기본-원칙)
2. [Layer별 점검 순서](#2-layer별-점검-순서)
3. [상황별 명령어 가이드](#3-상황별-명령어-가이드)
4. [명령어 상세 레퍼런스](#4-명령어-상세-레퍼런스)
5. [실전 트러블슈팅 시나리오](#5-실전-트러블슈팅-시나리오)

---

## 1. 트러블슈팅 기본 원칙

### 1.1 OSI 계층별 접근

장애가 발생하면 **아래 계층(물리/네트워크)부터** 위로 올라가며 점검합니다.

```
L7 (Application) - 어플리케이션 로직, HTTP 응답
L4 (Transport)   - TCP/UDP 연결 상태, 포트
L3 (Network)     - IP 라우팅, NAT, 방화벽
L2 (Data Link)   - MAC, ARP
L1 (Physical)    - 케이블, 인터페이스
```

### 1.2 진단 도구 선택 기준

| 알고 싶은 것 | 사용 도구 |
|------------|---------|
| 서비스가 동작하는지 | `curl`, `wget` |
| 호스트가 살아있는지 | `ping` (단, ICMP 차단 환경 주의) |
| 특정 포트가 열려있는지 | `nc`, `telnet`, `nmap` |
| 어디까지 도달하는지 | `traceroute`, `mtr` |
| 패킷이 실제로 오가는지 | `tcpdump` |
| DNS 정상 동작하는지 | `nslookup`, `dig` |
| 연결 상태/통계 | `ss`, `netstat` |
| 라우팅 경로 | `ip route`, `route` |

### 1.3 진단 순서 원칙

1. **결과 확인** (curl) → 증상이 무엇인가
2. **범위 좁히기** (ping, nc) → 어느 계층 문제인가
3. **경로 추적** (traceroute, mtr) → 어느 구간에서 문제인가
4. **패킷 분석** (tcpdump) → 정확히 무엇이 안 되는가
5. **시스템 상태** (ss, netstat, conntrack) → 시스템 자체 문제인가

---

## 2. Layer별 점검 순서

### 2.1 외부 통신 장애 표준 점검 흐름

```
[시작] 외부 통신 안 됨
    ↓
1. 서비스 응답 확인 (curl)
    ↓
2. DNS 해결 확인 (nslookup, dig)
    ↓
3. IP 직접 접속 확인 (curl -k IP)
    ↓
4. 포트 연결 확인 (nc, telnet)
    ↓
5. 경로 추적 (traceroute, mtr)
    ↓
6. 라우팅/NAT 확인 (ip route)
    ↓
7. 패킷 캡처 (tcpdump)
    ↓
8. 시스템 상태 (ss, netstat, conntrack)
    ↓
[원인 파악]
```

### 2.2 내부 통신 장애 점검 흐름

```
1. 동일 서브넷 통신 (ping 게이트웨이)
    ↓
2. ARP 테이블 확인 (ip neigh)
    ↓
3. 라우팅 테이블 (ip route)
    ↓
4. 방화벽/iptables 확인
    ↓
5. tcpdump로 패킷 흐름 확인
```

---

## 3. 상황별 명령어 가이드

### 3.1 "외부 사이트 접속이 안 돼요"

```bash
# 1. 일단 어떻게 안 되는지 확인
curl -v https://google.com

# 2. DNS 문제인지 분리
nslookup google.com
curl -k https://142.250.192.174   # 직접 IP

# 3. 포트가 막혔는지
nc -zv google.com 443

# 4. 어디까지 가는지
traceroute -T -p 443 google.com

# 5. 방화벽 응답 패턴 확인
sudo tcpdump -i eth0 -nn 'host google.com'
```

### 3.2 "통신이 가끔만 돼요" (간헐적 장애)

```bash
# 반복 테스트로 패턴 확인
for i in $(seq 1 20); do
  result=$(curl -o /dev/null -s -w "%{remote_ip}:%{remote_port} %{time_total}s" \
           --max-time 2 https://google.com)
  if [ $? -ne 0 ]; then
    echo "[$i] TIMEOUT"
  else
    echo "[$i] $result"
  fi
done

# mtr로 구간별 손실률 확인
mtr -r -c 100 google.com

# 시스템 자원 고갈 확인
cat /proc/sys/net/netfilter/nf_conntrack_count
cat /proc/sys/net/netfilter/nf_conntrack_max
ss -s
```

### 3.3 "연결은 되는데 느려요"

```bash
# 응답 시간 분리 측정
curl -o /dev/null -s -w "
DNS:        %{time_namelookup}s
Connect:    %{time_connect}s
TLS:        %{time_appconnect}s
Server:     %{time_starttransfer}s
Total:      %{time_total}s
" https://google.com

# 구간별 지연
mtr -r -c 50 google.com

# 대역폭 확인
iperf3 -c <대상서버>
```

### 3.4 "특정 포트가 열려있는지 확인하고 싶어요"

```bash
# 외부에서 내 서버 포트 확인 (외부 PC에서)
nc -zv <서버IP> 443
telnet <서버IP> 443
nmap -p 443 <서버IP>

# 내 서버에서 외부 포트 확인
nc -zv google.com 443

# 내 서버에서 LISTEN 중인 포트 확인
ss -tlnp
netstat -tlnp
```

### 3.5 "패킷이 실제로 도달하는지 확인하고 싶어요"

```bash
# 특정 호스트와의 모든 패킷
sudo tcpdump -i eth0 -nn 'host 142.250.192.174'

# 특정 포트만
sudo tcpdump -i eth0 -nn 'port 443'

# SYN 패킷만 (연결 시도)
sudo tcpdump -i eth0 -nn 'tcp[tcpflags] & tcp-syn != 0'

# SYN-ACK 패킷만 (연결 응답)
sudo tcpdump -i eth0 -nn 'tcp[tcpflags] & (tcp-syn|tcp-ack) == (tcp-syn|tcp-ack)'

# 파일로 저장 (Wireshark 분석용)
sudo tcpdump -i eth0 -nn -w capture.pcap 'host google.com'
```

### 3.6 "DNS가 이상해요"

```bash
# 기본 조회
nslookup google.com
dig google.com

# 특정 DNS 서버로 조회
dig @8.8.8.8 google.com
nslookup google.com 8.8.8.8

# 어떤 DNS 서버를 보고 있는지
cat /etc/resolv.conf

# 응답 시간 측정
dig google.com | grep "Query time"

# 역방향 조회
dig -x 142.250.192.174
```

### 3.7 "내 공인 IP가 뭐야?"

```bash
curl ifconfig.me
curl ipinfo.io/ip
curl checkip.amazonaws.com
```

### 3.8 "라우팅 경로 확인"

```bash
# 라우팅 테이블
ip route
route -n

# 특정 목적지로의 경로
ip route get 8.8.8.8

# 기본 게이트웨이만
ip route | grep default
```

---

## 4. 명령어 상세 레퍼런스

### 4.1 `curl` - HTTP/HTTPS 통신 테스트

**용도:** 웹 서비스 응답 확인, API 테스트, 응답 시간 측정

**자주 쓰는 옵션:**

| 옵션 | 의미 |
|------|------|
| `-v` | 상세 정보 (헤더, TLS 핸드셰이크) |
| `-k` | SSL 인증서 검증 무시 |
| `-o /dev/null` | 응답 본문 출력 안 함 |
| `-s` | 진행률 숨김 |
| `-w "포맷"` | 응답 정보 추출 (시간, 상태코드 등) |
| `--max-time N` | 전체 N초 타임아웃 |
| `--connect-timeout N` | 연결 N초 타임아웃 |
| `-I` | HEAD 요청만 |
| `-L` | 리다이렉트 따라감 |

**`-w` 출력 변수:**
- `%{http_code}` - HTTP 상태 코드
- `%{remote_ip}:%{remote_port}` - 실제 연결한 IP:포트
- `%{time_namelookup}` - DNS 조회 시간
- `%{time_connect}` - TCP 연결 시간
- `%{time_appconnect}` - TLS 핸드셰이크 시간
- `%{time_starttransfer}` - 첫 바이트 수신 시간
- `%{time_total}` - 전체 시간
- `%{size_download}` - 다운로드 크기

**예시:**
```bash
# 응답 시간 분석
curl -o /dev/null -s -w "%{http_code} %{time_total}s\n" https://google.com

# 헤더만 확인
curl -I https://google.com

# POST 요청
curl -X POST -H "Content-Type: application/json" -d '{"key":"val"}' https://api.example.com
```

---

### 4.2 `ping` - 호스트 도달 확인

**용도:** ICMP로 호스트 응답 확인, 패킷 손실률 측정

**주의:** 클라우드 환경에서 ICMP 차단된 경우가 많음. ping 안 된다고 통신 안 되는 건 아님.

| 옵션 | 의미 |
|------|------|
| `-c N` | N번만 보내고 종료 |
| `-i N` | N초 간격 |
| `-W N` | N초 응답 대기 |
| `-s N` | 패킷 크기 N 바이트 |
| `-f` | flood ping (root 권한, 부하 테스트) |

**예시:**
```bash
ping -c 10 google.com
ping -i 0.2 -c 100 google.com
ping -s 1472 -M do google.com   # MTU 테스트
```

---

### 4.3 `traceroute` / `mtr` - 경로 추적

**용도:** 패킷이 어떤 경로로 가는지, 어느 구간에서 막히는지 확인

**`traceroute` 옵션:**

| 옵션 | 의미 |
|------|------|
| `-T` | TCP 사용 (방화벽 우회용) |
| `-U` | UDP 사용 |
| `-I` | ICMP 사용 |
| `-p N` | 포트 N으로 |
| `-n` | DNS 역방향 조회 안 함 (빠름) |
| `-m N` | 최대 홉 수 |

**`mtr` 특징:**
- traceroute + ping을 합친 도구
- 실시간으로 각 홉의 손실률/지연시간 표시
- 간헐적 장애 분석에 매우 유용

```bash
traceroute -T -p 443 google.com   # HTTPS 경로 추적
mtr -r -c 100 google.com          # 100번 측정 후 리포트
mtr google.com                    # 실시간 모니터링
```

---

### 4.4 `nc` (netcat) - 포트 연결 테스트

**용도:** TCP/UDP 포트 연결 가능 여부 확인, 간단한 데이터 송수신

| 옵션 | 의미 |
|------|------|
| `-z` | 스캔 모드 (데이터 안 보냄) |
| `-v` | 상세 출력 |
| `-u` | UDP |
| `-l` | 리스닝 모드 (서버) |
| `-w N` | N초 타임아웃 |

```bash
nc -zv google.com 443          # 포트 열림 확인
nc -zv google.com 80-443       # 포트 범위 스캔
nc -zuv 8.8.8.8 53             # UDP 포트 확인
nc -lv 12345                   # 임시 서버 (수신 측)
nc <서버IP> 12345              # 연결 (송신 측)
```

---

### 4.5 `tcpdump` - 패킷 캡처

**용도:** 실제 송수신 패킷 캡처/분석. 다른 도구로 안 보이는 문제 추적의 최후 수단.

| 옵션 | 의미 |
|------|------|
| `-i <인터페이스>` | 캡처할 인터페이스 (`eth0`, `any`) |
| `-nn` | DNS/포트 이름 변환 안 함 (빠름, 명확) |
| `-w <파일>` | 파일로 저장 (Wireshark용) |
| `-r <파일>` | 저장된 파일 읽기 |
| `-c N` | N개 패킷만 캡처 |
| `-X` | 패킷 내용 hex/ASCII 출력 |
| `-s 0` | 패킷 전체 캡처 (기본 96바이트) |

**필터 표현식:**

```bash
host 1.2.3.4                                    # 호스트
src host 1.2.3.4                                # 출발지만
port 443                                        # 포트
'host 1.2.3.4 and port 443'                    # 조합
'not port 22'
'tcp[tcpflags] & tcp-syn != 0'                 # SYN
'tcp[tcpflags] & tcp-rst != 0'                 # RST
```

**실전 예시:**

```bash
# 외부 통신이 막히는지 확인
sudo tcpdump -i eth0 -nn 'host 142.250.192.174 and port 443'

# SYN-ACK 수신 여부 확인
sudo tcpdump -i eth0 -nn 'host google.com and tcp[tcpflags] & (tcp-syn|tcp-ack) != 0'

# DNS 쿼리 확인
sudo tcpdump -i eth0 -nn 'port 53'

# RST 패킷 추적
sudo tcpdump -i eth0 -nn 'tcp[tcpflags] & tcp-rst != 0'

# Wireshark 분석용 저장
sudo tcpdump -i eth0 -nn -w /tmp/capture.pcap 'host google.com'
```

---

### 4.6 `ss` / `netstat` - 연결 상태 확인

| 옵션 | 의미 |
|------|------|
| `-t` | TCP |
| `-u` | UDP |
| `-l` | LISTEN 상태만 |
| `-n` | 숫자로 표시 |
| `-p` | 프로세스 정보 |
| `-s` | 통계 요약 |

```bash
ss -tlnp                        # 모든 LISTEN 포트
ss -tan                         # 모든 TCP 연결
ss -tan state established       # ESTABLISHED만
ss -s                           # 통계
netstat -s | grep -i drop       # 드롭 확인
```

---

### 4.7 `ip` - 네트워크 설정 확인

```bash
ip addr                    # 인터페이스 정보
ip route                   # 라우팅 테이블
ip route get 8.8.8.8       # 특정 IP로의 경로
ip neigh                   # ARP 테이블
sudo ip link set eth0 up   # 인터페이스 활성화
```

---

### 4.8 `dig` / `nslookup` - DNS 조회

```bash
dig google.com              # 기본 A 레코드
dig +short google.com       # 결과만
dig google.com MX           # MX 레코드
dig @8.8.8.8 google.com    # 특정 DNS 서버로
dig +trace google.com       # 루트부터 추적
dig -x 142.250.192.174     # 역방향 조회
```

---

### 4.9 `conntrack` - 연결 추적 테이블

**용도:** NAT/방화벽 환경에서 연결 추적 상태 확인

```bash
cat /proc/sys/net/netfilter/nf_conntrack_count   # 현재 연결 수
cat /proc/sys/net/netfilter/nf_conntrack_max     # 최대 허용 수
sudo conntrack -L                                 # 추적 테이블 내용
sudo conntrack -S                                 # 통계
```

---

### 4.10 시스템 통계

```bash
netstat -s                                          # 네트워크 통계
ip -s link show eth0                                # 인터페이스 통계
netstat -s | grep -i "drop\|overflow\|fail\|retrans"  # 드롭 확인
cat /proc/net/sockstat                              # 소켓 메모리
```

---

## 5. 실전 트러블슈팅 시나리오

### 시나리오 1: 외부 통신 간헐적 실패 (NAT GW SNAT 포트 차단)

**증상:** `curl https://google.com` 약 50% 실패

```bash
# 1. 반복 테스트로 패턴 파악
for i in $(seq 1 20); do
  curl -o /dev/null -s -w "[$i] %{http_code} %{time_total}s\n" \
       --max-time 2 https://google.com || echo "[$i] TIMEOUT"
done

# 2. DNS 문제 배제
curl -k https://142.250.192.174 -o /dev/null -w "%{http_code}\n"

# 3. 시스템 자원 확인
cat /proc/sys/net/netfilter/nf_conntrack_count
ss -s

# 4. tcpdump로 패킷 흐름 확인 → SYN은 100% 나가는데 SYN-ACK은 50%만 돌아옴
sudo tcpdump -i eth0 -nn 'host 142.250.192.174'

# 5. 라우팅 확인
ip route get 142.250.192.174
```

**원인:** NAT GW 서브넷 NACL이 SNAT 응답 포트(1024-32767) 차단

**해결:** NACL Inbound에 1024-32767 허용 규칙 추가

---

### 시나리오 2: 특정 시간만 응답 지연

**증상:** 특정 시간대에만 응답이 느려짐

```bash
# 응답 시간 구간별 분석
curl -o /dev/null -s -w "
DNS:    %{time_namelookup}s
Connect: %{time_connect}s
TLS:    %{time_appconnect}s
Server: %{time_starttransfer}s
Total:  %{time_total}s
" https://api.example.com

# 지속 모니터링
mtr -r -c 1000 api.example.com
watch -n 1 'ss -s'
```

| 지연 구간 | 가능한 원인 |
|----------|-----------|
| `time_namelookup` 큼 | DNS 서버 지연 |
| `time_connect` 큼 | 네트워크 문제, 부하분산 |
| `time_appconnect` 큼 | TLS 처리 부하 |
| `time_starttransfer` 큼 | 서버 자체 문제 |

---

### 시나리오 3: 특정 외부 서비스만 접속 안 됨

**증상:** Google은 되는데 특정 API 서버만 안 됨

```bash
curl -v https://problem-api.com
dig problem-api.com
dig @8.8.8.8 problem-api.com
nc -zv problem-api.com 443
traceroute -T -p 443 problem-api.com
sudo tcpdump -i eth0 -nn 'host problem-api.com'
```

**가능한 원인:** 특정 IP 방화벽/NACL 차단, 해당 서버에서 IP 차단, DNS 캐시 오염

---

### 시나리오 4: 연결은 되는데 RST로 끊김

**증상:** 연결은 되지만 통신 중 갑자기 끊어짐

```bash
# RST 패킷 추적 (어느 쪽에서 보냈는지 출발지 IP 확인)
sudo tcpdump -i eth0 -nn 'tcp[tcpflags] & tcp-rst != 0'

# 재전송/오류 확인
netstat -s | grep -i "retrans\|reset"
```

**가능한 원인:** IDS/IPS 차단, 서버 측 idle timeout, 방화벽 세션 만료, 부하분산기 health check 실패

---

## 6. 트러블슈팅 체크리스트

외부 통신 장애 발생 시:

- [ ] curl로 증상 재현 가능한가?
- [ ] 모든 외부 사이트가 안 되는가, 특정 사이트만 안 되는가?
- [ ] DNS는 정상 동작하는가?
- [ ] IP 직접 접속은 되는가?
- [ ] 다른 포트는 되는가?
- [ ] traceroute/mtr로 어디까지 도달하는가?
- [ ] tcpdump로 SYN/SYN-ACK/RST 패턴은?
- [ ] conntrack 테이블은 정상 범위인가?
- [ ] 라우팅 테이블은 정상인가?
- [ ] 방화벽(iptables/ACG/NACL) 정책은 정상인가?
- [ ] 클라우드 환경의 경우 NAT GW, 보안 그룹 점검했는가?

---

## 7. 참고 사항

### ICMP 차단 환경

클라우드/엔터프라이즈 환경에서 ICMP가 차단된 경우가 많습니다.
- `ping` 안 된다고 호스트 죽은 것이 아님
- `traceroute -I` (ICMP) 안 되면 `-T` (TCP)로 시도
- 실제 서비스 통신은 TCP/UDP이므로 해당 프로토콜로 테스트

### 운영 환경 주의사항

- **`tcpdump`는 부하 발생 가능** → 필터 좁게 설정 (`host`, `port`)
- **`mtr`는 지속 ping이라 부하 가능** → `-c` 옵션으로 횟수 제한
- **`nmap`은 보안 알람 유발 가능** → 운영 환경에서 함부로 쓰지 말 것
- **로그 캡처는 디스크 차면 장애 유발** → 적절한 시점에 종료

### tcpdump 권한 설정

```bash
sudo setcap cap_net_raw,cap_net_admin=eip $(which tcpdump)
```
