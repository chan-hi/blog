---
title: "Chapter 15: 교통 정리 전문가 — Load Balancer"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, load-balancer, nlb, alb, proxy-lb, dsr]
description: "NCP NCE 자격증 대비 학습 자료 - NLB, Proxy Load Balancer, ALB 구조와 알고리즘 비교"
---
> **학습 목표**: ALB/NLB/Network Proxy LB의 차이, Listener/Target Group/알고리즘, DSR/SSL Offload를 완벽히 구분한다.

---

## 이야기: 주문이 폭발했다

클라우드빵집 TV 광고 후 동시 접속자가 10만 명으로 폭증했다.
서버 1대로는 감당이 불가능했다.

> "서버를 10대 늘렸어. 근데 고객 요청을 어떻게 10대에 균등하게 나눠?"

재현이 말했다.

> "로드밸런서(Load Balancer). 앞에서 교통 정리를 해주는 장치야.
> 들어오는 요청을 여러 서버에 분산시켜줘."

---

## 🔑 핵심 개념 1: 로드밸런서 구성 요소

```
[로드밸런서 구조]

클라이언트
    ↓
[Listener] — 어떤 포트/프로토콜로 받을지 결정
    ↓
[Rule] — 어느 Target Group으로 보낼지 규칙
    ↓
[Target Group] — 실제 서버들의 그룹
    ├─ 서버1 (10.0.1.10:8080)
    ├─ 서버2 (10.0.1.11:8080)
    └─ 서버3 (10.0.1.12:8080)
```

| 구성 요소 | 역할 |
|---------|------|
| **Listener** | 클라이언트로부터 오는 트랜잭션을 연결 요청의 프로토콜과 포트 번호 체크 후, 조건이 맞는 타겟으로 포워딩 |
| **Rule** | 어느 Target Group에 전달할지 판단하는 기준이 되는 규칙 (규칙 조건/규칙 액션) |
| **Target Group** | 리스너가 전달한 요청을 처리하기 위한 부하 분산 대상들의 모임 (동일 VPC 내 서버) |

### Target Group 상세

- **대상 범위**: 동일 VPC 내에 있는 서버들에 대해 타겟 그룹 생성 가능
- **공유 제한**: 타겟 그룹 안에 있는 서버를 다른 타겟 그룹에도 속하게 할 수 있으나, **타겟 그룹을 다수의 로드밸런서에 연결할 수는 없음**
- **계층 구분**: 서비스를 수행하는 대상의 프로토콜에 따라 L4와 L7으로 구분
- **Health Check**: 주기 5~300초 및 임계값 설정 가능
- **기본 알고리즘**: Round Robin
- **변경 시점**: 알고리즘, Sticky Session, Proxy Protocol 설정 변경은 **생성 이후**에 진행

**Target Group 프로토콜과 LB 매핑:**

| 프로토콜 | 로드밸런서 |
|---------|-----------|
| TCP, UDP | Network Load Balancer |
| Proxy_TCP | Network Proxy Load Balancer |
| HTTP, HTTPS | Application Load Balancer |
| IP | Inline Load Balancer |

---

## 🔑 핵심 개념 2: 부하 분산 알고리즘 3가지 (⭐ 빈출!)

| 알고리즘 | 방식 | 특징 |
|---------|------|------|
| **Round Robin** | 서버에 순차적으로 1개씩 배분 | 가장 기본적, 단순 순환 |
| **Least Connection** | 현재 연결 수가 가장 적은 서버에 배분 | 처리 시간이 다를 때 유리 |
| **Source IP Hash** | 클라이언트 IP 해시 → 항상 같은 서버로 | 동일 클라이언트는 항상 같은 서버 |

> 💡 **Round Robin**: "1번, 2번, 3번, 1번, 2번..." 차례대로  
> **Least Connection**: 가장 한가한 서버에  
> **Source IP Hash**: 같은 손님은 항상 같은 직원에게

- **NLB**: Hash, Round Robin 2가지만 제공
- **ALB / Network Proxy LB**: 3가지 모두 제공

### LB 이론상 구성 방식

로드밸런서의 내부 동작 방식은 크게 세 가지로 구분된다.

| 방식 | 설명 |
|------|------|
| **NAT** | 패킷의 IP 주소를 변환하여 전달 |
| **DR (Direct Return)** | 패킷을 그대로 서버에 전달, 응답은 서버가 직접 클라이언트에 반환 (DSR의 기반) |
| **Proxy** | LB가 클라이언트-서버 간 TCP 연결을 직접 중계 (두 연결로 분리) |

---

## 🔑 핵심 개념 3: 로드밸런서 3종류 비교 (⭐⭐ 최빈출!)

| 기능 | **Network LB** | **Network Proxy LB** | **Application LB** |
|------|--------------|--------------------|--------------------|
| **프로토콜** | **TCP, UDP** | **TCP, TLS** | **HTTP, HTTPS** |
| **OSI 계층** | **L4** | L4 (Proxy) | **L7** |
| **상태 확인(Health Check)** | ✅ | ✅ | ✅ |
| **로깅** | ❌ | ✅ | ✅ |
| **DSR** | **✅** | ❌ | ❌ |
| **같은 인스턴스 여러 포트 분산** | ❌ | ❌ | **✅** |
| **HTTP 2.0 지원** | N/A | N/A | **✅** |
| **경로 기반 라우팅** | N/A | N/A | **✅** |
| **SSL 오프로드** | ❌ | **✅** | **✅** |
| **고정 세션(Sticky Session)** | **✅** | ❌ | **✅** |
| **Multi Zone** | ❌ | ✅ | ✅ |
| **분산 알고리즘** | Hash, Round Robin | 3가지 모두 | 3가지 모두 |
| **CPS (초당 연결)** | Dynamic Sizing | 30,000 ~ 120,000 | 30,000 ~ 120,000 |

---

### 각 LB 특징 상세

#### Network Load Balancer (NLB) — L4
- TCP/UDP 프로토콜, 고성능 처리 특화
- **Dynamic Sizing**: 트래픽에 따라 자동으로 성능 조절 (CPS 동적 확장)
- **DSR (Direct Server Return)**: LB를 거치지 않고 서버가 클라이언트에 직접 응답 → 효율 극대화
- 클라이언트 실제 IP가 서버 로그에 그대로 기록됨 (SNAT 없음)
- 알고리즘은 Hash, Round Robin 2가지만 지원

#### Network Proxy Load Balancer — L4 Proxy
- TCP 세션 관리가 필요한 TCP 기반 애플리케이션에 적합
- Classic LB와 유사한 방식
- **SSL 오프로드 지원**: TLSv1 / TLSv1.1 / TLSv1.2 프로토콜 중 선택 가능, Certificate Manager 서비스와 연동하여 인증서 관리
- **CPS**: Small/Medium/Large/Extra Large에 따라 각각 30,000 / 60,000 / 90,000 / 120,000 보장
- ALB와 동일한 3가지 부하 분산 알고리즘 적용 가능
- **TCP 세션 관리**: LB가 클라이언트-서버 간 TCP 연결을 직접 중계 (두 연결로 분리)

```
[NLB - pass-through]
클라이언트 ──────────────── 서버  (TCP 연결 1개)

[NPLB - Proxy]
클라이언트 ── TCP연결1 ── NPLB ── TCP연결2 ── 서버
```

- Proxy 방식이므로 **서버 입장의 Source IP = LB IP** (클라이언트 IP가 아님)
- 원본 클라이언트 IP를 확인하려면 **Proxy Protocol** 활성화 필요

#### Application Load Balancer (ALB) — L7
- HTTP/HTTPS를 사용하는 웹 애플리케이션에 보다 유연한 구성 가능
- **고정 IP 제공**
- HTTP/HTTPS 헤더 분석 가능 → 다양한 L7 분기 처리 지원:
  - **Host Header 기반 분기** 처리
  - **URL Path Pattern 기반 분기** 처리
  - **가중치 기반 분기** 처리
  - **Redirection 응답** 처리
- 같은 서버의 여러 포트로 분산 가능
- HTTP 2.0 지원
- **CPS**: Small/Medium/Large/Extra Large에 따라 각각 30,000 / 60,000 / 90,000 / 120,000 보장

---

## 🔑 핵심 개념 4: DSR (Direct Server Return)

```
[일반 방식]
클라이언트 → LB → 서버 → LB → 클라이언트
(응답도 LB를 거침 → LB 부하 높음)

[DSR 방식]
클라이언트 → LB → 서버 → 클라이언트 (직접!)
(응답은 서버에서 직접 → LB 부하 최소)
```

> DSR은 **Network LB에서만** 지원!  
> 대용량 응답 트래픽(동영상 스트리밍 등)에 유리

### DSR이 동작하는 두 가지 조건

DSR은 "알아서 생기는 현상"이 아니라 **의도적으로 설계된 아키텍처**다.

| 조건 | 설명 |
|------|------|
| **① LB가 패킷 pass-through (SNAT 없음)** | 패킷을 수정하지 않고 그대로 전달 |
| **② 서버가 VIP를 자신의 IP로 인식** | 서버의 loopback 인터페이스에 LB의 VIP 등록 |

- ①만으로는 서버가 응답 패킷의 목적지를 어디로 보낼지 알 수 없다
- ②가 있어야 서버가 VIP로 들어온 패킷을 "내 요청"으로 처리하고, 직접 응답 생성 가능
- 응답을 클라이언트에게 직접 보내는 것은 ①+②의 **자연스러운 결과**

### DSR과 Source IP 보존의 관계

> DSR이라서 Source IP가 보존되는 게 아니라, **SNAT를 안 하기 때문에** 클라이언트 IP가 그대로 서버에 전달된다.

- **Source IP 보존 여부** = SNAT 여부에 달린 문제
- DSR은 응답 트래픽 경로의 특성 (별개 개념)
- NLB는 "SNAT 없음"과 "DSR"이 함께 나타나지만, 인과관계는 다름

---

## 🔑 핵심 개념 5: 클라이언트 IP 확인 방법

서버 입장에서 실제 클라이언트 IP를 확인하는 방법은 LB 종류마다 다르다.

| LB 종류 | 서버가 보는 Source IP | 클라이언트 IP 확인 방법 |
|--------|---------------------|----------------------|
| **NLB** | 클라이언트 IP (그대로) | 별도 설정 불필요 |
| **NPLB** | LB IP | **Proxy Protocol** 활성화 |
| **ALB** | LB IP | **X-Forwarded-For** 헤더 |

### Proxy Protocol

TCP 레벨에서 원본 클라이언트 IP를 전달하는 방식. HTTP 헤더가 없는 TCP/TLS 환경에서 사용.

```
# TCP 연결 맨 앞에 클라이언트 정보를 텍스트로 붙여 전달
PROXY TCP4 125.209.237.10 125.209.192.12 43321 80\r\n
           ↑ 클라이언트IP  ↑ LB IP       ↑클라포트 ↑서버포트
```

> ⚠️ **양쪽 모두 설정 필요**: LB에서 Proxy Protocol 활성화 + 애플리케이션(nginx 등)에서 수신 설정 활성화  
> 한쪽만 켜면 서버가 `PROXY TCP4 ...` 텍스트를 HTTP 요청으로 오인해 오류 발생

### X-Forwarded-For 헤더

ALB 환경에서 애플리케이션이 실제 클라이언트 IP를 식별하는 방법.

```php
// Proxy를 이용할 경우 Proxy에 들어있는 클라이언트의 IP
$_SERVER['HTTP_X_FORWARDED_FOR'];

// 일반적인 클라이언트 IP (IP 헤더값)
$_SERVER['REMOTE_ADDR'];
```

---

## 🔑 핵심 개념 6: SSL Redirect (ALB 활용 패턴)

SSL을 이용할 경우, HTTP로 들어온 요청을 HTTPS로 리다이렉트하는 방법이 두 가지 있다.

### 방법 1: 서버 측 Redirect

```
[SSL 이용 시 포트 흐름]
클라이언트 → LB (443) → 서버 (8080)  ← HTTPS 처리

클라이언트 → LB (80) → 서버 (8080)  ← HTTP로 접근 시
           서버에서 443으로 Redirect
```

- LB는 TCP 443 포트로 받고 서버 8080 포트로 전달
- 서버에서 80 포트로 들어온 접근을 443으로 리다이렉트 처리
- 서버에서 8080 포트도 웹 서버에서 Listen하도록 설정 필요

### 방법 2: X-Forwarded-Proto 헤더 활용

- `X-Forwarded-Proto` 헤더를 이용하여 클라이언트가 연결에 사용한 프로토콜을 식별
- HTTP로 온 경우 HTTPS로 Redirect 처리

---

## 🔑 핵심 개념 7: Idle Timeout (유휴 제한 시간)

로드밸런서가 클라이언트와 서버 간 연결을 유지하는 최대 시간. 클라이언트가 일정 시간 동안 요청을 보내지 않으면 해당 연결을 자동으로 종료한다.

### Idle Timeout이 중요한 이유

| 효과 | 설명 |
|------|------|
| **서버 리소스 절약** | 클라이언트가 너무 오래 연결을 유지하면 서버 리소스 낭비 |
| **동시 접속 수 증가** | 불필요한 연결을 정리해 최대 동시 접속 가능 수 증가 |

### Idle Timeout을 늘려야 할 상황

- **실시간 스트리밍** (WebSocket, SSE)
- **대용량 파일 업로드** (멀티파트 업로드)
- **API 서버에서 긴 실행 요청** (Long-polling 등)

---

## 🔑 핵심 개념 8: Sticky Session (고정 세션)

로드밸런서를 통해 들어오는 사용자의 요청을 **항상 같은 서버로 보내는 방식**.

일반적으로 로드밸런서는 요청을 여러 서버에 분산하지만, Sticky Session을 사용하면 특정 사용자는 같은 서버로 계속 연결이 유지된다.

**적용 가능 조건**: Target Group의 프로토콜이 TCP / UDP / HTTP / HTTPS인 경우

### Sticky Session이 필요한 이유

| 상황 | 설명 |
|------|------|
| **서버 측 세션 저장** | 로그인 정보, 장바구니 데이터가 서버에 저장되는 경우. 다른 서버로 연결되면 세션이 끊길 수 있음 |
| **서버 캐시 효율** | 동일한 사용자의 요청을 같은 서버로 보내면 서버 캐시 HIT 가능성 증가 (개인화된 콘텐츠 추천) |

```
[Sticky Session 적용 전]              [Sticky Session 적용 후]
사용자 → LB → 서버1 (첫 번째 요청)   사용자 → LB → 서버1 (첫 번째 요청)
사용자 → LB → 서버2 (두 번째 요청)   사용자 → LB → 서버1 (두 번째 요청, 고정)
사용자 → LB → 서버3 (세 번째 요청)   사용자 → LB → 서버1 (세 번째 요청, 고정)
```

> NLB는 Sticky Session 지원 / Network Proxy LB는 **미지원** / ALB는 지원

---

## 🔑 핵심 개념 9: 포트 설정 규칙

> ⚠️ **시험 포인트**  
> 여러 규칙을 동시에 설정할 때:
> - **LB 포트**: 서로 **다르게** 설정해야 함
> - **서버 포트**: 동일하게 설정해도 무방

```
[예시]
규칙1: LB 포트 80  → 서버 포트 8080 (O)
규칙2: LB 포트 443 → 서버 포트 8080 (O)
       ↑ LB 포트 다름  ↑ 서버 포트 같아도 OK
```

---

## 🔑 핵심 개념 10: 모니터링

| LB 종류 | 모니터링 항목 수 | 항목 내용 |
|--------|--------------|---------|
| Network Load Balancer | **5가지** | ConcurrentConnection, 초당 Connection, Traffic In, (Un)Available hosts 등 |
| Application Load Balancer | **6가지** | NLB 5가지 + **Traffic Out** 포함 |
| 수집 주기 | **1분, 5분, 2시간, 1일** | 기간 선택에 따라 수집 주기 변동 |

---

## 📝 시험 대비 체크리스트

- [ ] Listener: 프로토콜/포트 체크 → Target Group으로 라우팅
- [ ] Target Group: 다수 LB에 연결 불가 / 알고리즘·Sticky·Proxy Protocol 변경은 생성 이후
- [ ] Round Robin: 순차 배분 / Least Connection: 최소 연결 / Source IP Hash: IP 해시
- [ ] NLB: TCP/UDP, L4, **DSR 지원**, 로깅 ❌, Source IP = 클라이언트 IP (그대로), 알고리즘 2가지
- [ ] Network Proxy LB: TCP/TLS, SSL 오프로드 ✅, DSR ❌, Source IP = LB IP, CPS 30,000~120,000
- [ ] ALB: HTTP/HTTPS, L7, URL 경로/Host Header/가중치/Redirect 기반, 같은 서버 여러 포트 ✅, 고정 IP, CPS 30,000~120,000
- [ ] DSR: **NLB만** 지원 / 조건: ① LB pass-through(SNAT 없음) + ② 서버가 VIP를 자신의 IP로 인식
- [ ] Source IP 보존은 DSR 때문이 아니라 **SNAT 없음** 때문
- [ ] 클라이언트 IP 확인: NLB=그대로, NPLB=Proxy Protocol, ALB=X-Forwarded-For
- [ ] Proxy Protocol: LB + 앱 **양쪽 모두** 설정 필요
- [ ] Idle Timeout: 불필요한 연결 정리 → 리소스 절약, 동시 접속 수 증가
- [ ] Sticky Session: NLB/ALB 지원, NPLB 미지원 / 세션 유지가 필요한 서비스에 사용
- [ ] LB 포트: 규칙마다 **다르게** / 서버 포트: 같아도 OK
- [ ] ALB/NPLB CPS: 30,000 / 60,000 / 90,000 / 120,000 (Small/Medium/Large/Extra Large)
- [ ] NLB CPS: Dynamic Sizing
- [ ] 모니터링 수집 주기: 1분, 5분, 2시간, 1일
- [ ] ALB 모니터링: 6가지 (NLB 5가지 + Traffic Out)

---

## 🧠 암기 핵심 문장

> "**NLB = TCP/UDP + DSR + L4 / ALB = HTTP + 경로기반 + L7**"  
> "**DSR은 NLB만, SSL 오프로드는 Proxy+ALB**"  
> "**Source IP 보존 = SNAT 없음 / Proxy Protocol = TCP에서 클라이언트 IP 전달**"  
> "**Sticky Session = NLB/ALB 지원, NPLB 미지원 / Idle Timeout = 불필요한 연결 정리**"

---

*다음 챕터: `16_로드밸런서_심화.md` → Sticky Session, Idle Timeout, X-Forwarded-For*
