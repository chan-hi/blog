---
title: "NCP NAT Gateway 외부 통신 간헐적 패킷 손실 트러블슈팅"
date: 2026-04-20 00:00:00 +0900
categories: [NCP, Troubleshooting]
tags: [ncp, nat-gateway, nacl, snat, troubleshooting, packet-loss]
description: "NCP NAT Gateway 사용 시 간헐적 패킷 손실(46~70%)의 원인인 NACL Stateless 특성과 SNAT 포트 범위 불일치 문제 분석 및 해결"
---

# NCP NAT Gateway 외부 통신 간헐적 패킷 손실 트러블슈팅 보고서

- **작성일:** 2026-04-20
- **대상 서버:** web-server-01 (10.0.10.9)
- **관련 자원:** nat-gw2 (NAT Gateway, 공인IP x.x.x.x)

---

## 1. 장애 개요

### 1.1 현상

서버에서 외부 인터넷으로 HTTPS 통신 시 약 **46~70%의 패킷 손실** 발생.

```bash
[1] 142.250.192.174:443 0.164365s   ← 성공
[2] 142.250.192.174:443 0.159666s   ← 성공
[3] TIMEOUT
[4] TIMEOUT
[5] TIMEOUT
[6] TIMEOUT
[7] TIMEOUT
[8] TIMEOUT
[9] TIMEOUT
[10] TIMEOUT
[11] TIMEOUT
[12] 142.250.192.174:443 0.156925s
[13] TIMEOUT
[14] 142.250.192.174:443 0.159902s
[15] TIMEOUT
...
```

### 1.2 특징

- 성공 시 응답속도는 정상 (~0.16s)
- 실패 시 연결 자체가 수립되지 않음 (TCP SYN 송출되나 SYN-ACK 미수신)
- 특정 목적지가 아닌 **Google, Cloudflare, Naver, GitHub** 등 전체 외부 구간에서 동일 패턴
- `curl ifconfig.me`는 완전 먹통

---

## 2. 환경 구성

```
서버 web-server-01 (10.0.10.9)
    ↓ default route via 10.0.10.1
NAT Gateway nat-gw2
    내부 IP : 10.0.32.6
    공인 IP : x.x.x.x
    전용 서브넷 : 10.0.32.x
    ↓
인터넷
```

- NAT Gateway에 연결된 서버: 3대
- NAT Gateway는 전용 서브넷(10.0.32.x) 사용
- 서버 서브넷(10.0.10.x)과 NAT GW 서브넷(10.0.32.x)에 동일한 NACL 적용

### 2.1 ACG 설정 (요약)

**Inbound:**

| 프로토콜 | 접근 소스 | 허용 포트 | 메모 |
|---------|----------|----------|------|
| TCP | x.x.x.x/32 | 22 | office |
| TCP | 10.0.0.0/16 | 80 | Internal HTTP for ALB |
| TCP | 10.0.0.0/16 | 22 | Internal SSH for ASG |
| TCP | 10.1.0.0/16 | 1-65535 | |
| TCP | 0.0.0.0/0 | 443 | HTTPS |
| TCP | 0.0.0.0/0 | 80 | HTTP |
| ... | | | |

**Outbound:**

| 프로토콜 | 목적지 | 허용 포트 |
|---------|-------|----------|
| TCP | 0.0.0.0/0 | 1-65535 |
| TCP | 0.0.0.0/0 | 443 |
| TCP | 0.0.0.0/0 | 80 |
| UDP | 0.0.0.0/0 | 53 |
| ... | | |

### 2.2 NACL 설정 (장애 발생 시점)

**Inbound:**

| 우선순위 | 프로토콜 | 접근 소스 | 포트 | 허용여부 |
|---------|---------|----------|------|---------|
| 0 | TCP | x.x.x.x/32 | 1-65535 | 허용 |
| 1 | TCP | 10.0.0.0/16 | 1-65535 | 허용 |
| 2 | TCP | 0.0.0.0/0 | 22 | 허용 |
| 3 | TCP | 0.0.0.0/0 | 80 | 허용 |
| 4 | TCP | 0.0.0.0/0 | 443 | 허용 |
| 5 | TCP | 10.1.0.0/16 | 1-65535 | 허용 |
| **10** | **TCP** | **0.0.0.0/0** | **32768-65535** | **허용** |
| 11 | UDP | 0.0.0.0/0 | 32768-65535 | 허용 |
| 197 | TCP | 0.0.0.0/0 | 1-65535 | 차단 |
| 198 | UDP | 0.0.0.0/0 | 1-65535 | 차단 |
| 199 | ICMP | 0.0.0.0/0 | | 차단 |

**Outbound:**

| 우선순위 | 프로토콜 | 목적지 | 포트 | 허용여부 | 비고 |
|---------|---------|-------|------|---------|------|
| 0 | TCP | 222.xxx.xxx.xx/32 | 1-65535 | 허용 | |
| 1 | TCP | 10.0.0.0/16 | 1-65535 | 허용 | |
| 2 | TCP | 0.0.0.0/0 | 80 | 허용 | |
| 3 | TCP | 0.0.0.0/0 | 443 | 허용 | |
| 4 | UDP | 0.0.0.0/0 | 53 | 허용 | |
| 5 | TCP | 10.1.0.0/16 | 1-65535 | 허용 | |
| 10 | TCP | 0.0.0.0/0 | 32768-65535 | 허용 | |
| 11 | UDP | 0.0.0.0/0 | 32768-65535 | 허용 | |
| 20 | ICMP | 10.0.0.0/16 | | 허용 | |
| 197 | TCP | 0.0.0.0/0 | 1-65535 | 차단 | |
| 198 | UDP | 0.0.0.0/0 | 1-65535 | 차단 | |
| 199 | ICMP | 0.0.0.0/0 | | 차단 | |

---

## 3. 트러블슈팅 과정

### 3.1 1차 검토 결과 (NAT 가능성 배제 및 일반 점검)

| 점검 항목 | 결과 | 판단 |
|----------|------|------|
| 라우팅 테이블 | NAT Gateway 정상 설정 | 이상 없음 |
| ACG | Outbound 전체 허용, Stateful 정상 | 이상 없음 |
| NACL | Inbound/Outbound 규칙 검토 | **이상 없음 (오판)** |
| 서버 내부 | 추가 점검 진행 예정 | - |
| NAT Gateway 구간 | NCP 측 확인 요청 접수 | - |

### 3.2 서버 내부 점검

**conntrack 테이블 확인:**
```bash
$ cat /proc/sys/net/netfilter/nf_conntrack_count
51
$ cat /proc/sys/net/netfilter/nf_conntrack_max
262144
```
→ 세션 테이블 사용량 매우 낮음, 고갈 아님

**netstat 드롭 확인:**
```bash
$ netstat -s | grep -i "drop\|overflow\|fail"
0 input ICMP message failed
0 ICMP messages failed
2 failed connection attempts
```
→ 커널 레벨 패킷 드롭 없음

**DNS 문제 배제:**
```bash
$ curl -o /dev/null -s -w "%{time_total}s\n" --max-time 2 https://142.250.192.174 -k
0.155095s
```
→ IP 직접 접속 성공, DNS 무관

### 3.3 2차 분석: 원인 확인

#### 핵심 개념 1: SNAT 포트

서버는 사설 IP이므로 인터넷과 직접 통신 불가. NAT Gateway가 사설 IP를 자신의 공인 IP로 변환하여 외부와 통신함. 이를 **SNAT(Source NAT)** 라고 함.

여러 서버가 NAT GW를 공유하므로, 외부 응답이 어느 서버로 가야 하는지 구분하기 위해 **SNAT 포트** 사용:

```
[요청 변환]
서버:    10.0.10.9:54321    →  Google:443
NAT GW:  x.x.x.x:15000     →  Google:443   ← SNAT 포트 = 15000

[응답 역변환]
Google:443 → x.x.x.x:15000
NAT GW가 SNAT 포트(15000)를 보고 → 10.0.10.9:54321로 전달
```

**NCP NAT Gateway는 SNAT 포트를 1024~65535 범위에서 연결마다 동적으로 임의 배정함.**

#### 핵심 개념 2: NACL Stateless

| 구분 | 방식 | 응답 패킷 처리 |
|------|------|--------------|
| Security Group (ACG) | Stateful | 요청한 연결의 응답 자동 허용 |
| Network ACL (NACL) | Stateless | 응답 패킷도 명시적 허용 규칙 필요 |

NACL은 연결 상태를 추적하지 않고 **포트 번호만으로 허용/차단을 판단**함.

#### 문제 발생 메커니즘

응답 패킷 흐름:
```
Google:443 → 49.50.138.192:SNAT_PORT → [NACL Inbound 판단]
```

기존 NACL Inbound 규칙으로 판단 시:

| SNAT 포트 | 적용 규칙 | 결과 |
|----------|----------|------|
| 32768~65535 | 우선순위 10번 (허용) | ✅ 통신 성공 |
| 1024~32767 | 우선순위 197번 (차단) | ❌ 타임아웃 |

NCP NAT GW가 SNAT 포트를 1024~65535 전체에서 임의 배정하므로, **약 절반의 연결이 응답 단계에서 차단**되어 간헐적 패킷 손실로 나타남.

---

## 4. 해결 방법

### 4.1 NAT GW 서브넷 NACL Inbound 규칙 추가

**경로:** NCP 콘솔 → VPC → Network ACL → NAT GW 서브넷 NACL → Inbound 규칙

| 우선순위 | 프로토콜 | 접근 소스 | 포트 | 허용여부 | 메모 |
|---------|---------|----------|------|---------|------|
| **9** | **TCP** | **0.0.0.0/0** | **1024-32767** | **허용** | 외부 통신 응답 포트 허용 |

기존 규칙(우선순위 10번, 32768-65535)은 변경 없이 유지.

### 4.2 적용 결과

규칙 추가 후 외부 통신 100% 정상화 확인.

---

## 5. 보안 검토

### 5.1 본 규칙의 보안 영향

추가 규칙은 외부에서 서버로 **신규 접속을 허용하는 것이 아님**. NAT Gateway가 먼저 요청한 세션의 **응답 패킷만 허용**하는 구성.

추가 보안 근거:
1. NAT GW 전용 서브넷에는 실제 서비스 서버가 존재하지 않음
2. NAT GW 자체는 외부 노출 서비스가 아니므로 직접 공격 대상이 아님
3. 실제 서버 보안은 서버 서브넷의 NACL과 ACG에서 제어됨

### 5.2 NAT 동작 필수 조건

NCP NAT Gateway는 1024~65535 범위에서 SNAT 포트를 임의 배정하도록 **NCP 측에서 설계됨**. 변경 불가.

따라서 NAT GW가 위치한 서브넷의 NACL은 해당 범위 전체에 대한 Inbound 허용이 **기술적으로 필수**임.

### 5.3 서버 서브넷 NACL은 변경 불필요

서버 서브넷의 NACL은 그대로 유지 가능:

| 구간 | 사용 포트 범위 | 기존 NACL 규칙 |
|------|--------------|--------------|
| Linux 서버 ephemeral 포트 | 32768-60999 (커널 기본) | 규칙 10번 (32768-65535)으로 커버 ✅ |
| NAT GW SNAT 포트 | 1024-65535 (NCP 결정) | 규칙 10번만으로 부족, 1024-32767 추가 필요 |

Linux는 ephemeral 포트로 32768 미만을 사용하지 않으므로, 서버 서브넷에 1024-32767을 열 필요 없음.

---

## 6. 재발 방지 권고

NCP 환경에서 NAT Gateway 사용 시 아래 원칙 준수:

1. **NACL은 Stateless** → 아웃바운드 요청에 대한 응답 포트(ephemeral port)를 Inbound에 명시적으로 허용해야 함
2. **NCP NAT GW SNAT 포트 범위는 1024-65535** → NAT GW 서브넷 NACL Inbound에 해당 범위 전체 허용 필수
3. **NAT GW 서브넷 NACL은 최소 구성 권장** → 보안 통제는 서버 서브넷의 NACL/ACG에서 수행
4. **Stateful이 필요한 경우 ACG 활용** → 연결 상태 추적이 필요한 보안 정책은 NACL이 아닌 ACG로 구현

---

## 7. 트러블슈팅 명령어 모음

### 외부 통신 테스트
```bash
for i in $(seq 1 20); do
  result=$(curl -o /dev/null -s -w "%{remote_ip}:%{remote_port} %{time_total}s" \
           --max-time 2 https://google.com)
  if [ $? -ne 0 ]; then
    echo "[$i] TIMEOUT"
  else
    echo "[$i] $result"
  fi
done
```

### conntrack 확인
```bash
cat /proc/sys/net/netfilter/nf_conntrack_count
cat /proc/sys/net/netfilter/nf_conntrack_max
ss -s
```

### 패킷 드롭 확인
```bash
netstat -s | grep -i "drop\|overflow\|fail"
```

### 라우팅 확인
```bash
ip route | grep default
traceroute -T -p 443 google.com
```

### SSH 접속 (Bastion 경유)
```bash
# ssh-agent 기반 ProxyJump
eval $(ssh-agent -s)
ssh-add /path/to/key.pem
ssh -A -i /path/to/key.pem -o StrictHostKeyChecking=no \
    -J root@{bastion-ip} root@10.0.10.9
```

---

## 8. 참고 사항

- 본 분석에서 1차 검토 시 NACL "이상 없음" 판단은 규칙 자체의 문법적 오류 검토에 한정된 것이었음. NAT GW의 SNAT 포트 동작 특성과 NACL Stateless 특성을 함께 고려한 재분석에서 원인이 확인됨.
- NCP NAT Gateway의 SNAT 포트 범위는 NCP 공식 문서/관제팀 확인 필요 (현재는 동작 관찰 기반 추정).
