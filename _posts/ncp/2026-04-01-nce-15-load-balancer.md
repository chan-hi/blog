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
| **Listener** | 요청을 받아들이는 입구. 프로토콜과 포트 번호를 체크 |
| **Rule** | 어느 Target Group에 전달할지 판단하는 기준 규칙 |
| **Target Group** | 동일 VPC 내 부하분산 대상 서버들의 그룹 |

**Target Group 기본 설정:**
- 부하 분산 알고리즘: 기본 **Round Robin**
- Health Check 주기: **5~300초**
- Health Check 임계값: 설정 가능

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

---

## 🔑 핵심 개념 3: 로드밸런서 3종류 비교 (⭐⭐ 최빈출!)

| 기능 | **Network LB** | **Network Proxy LB** | **Application LB** |
|------|--------------|--------------------|--------------------|
| **프로토콜** | **TCP, UDP** | **TCP, TLS** | **HTTP, HTTPS** |
| **OSI 계층** | **L4** | L4 (Proxy) | **L7** |
| **Health Check** | ✅ | ✅ | ✅ |
| **로깅** | ❌ | ✅ | ✅ |
| **DSR** | **✅** | ❌ | ❌ |
| **같은 인스턴스 여러 포트 분산** | ❌ | ❌ | **✅** |
| **HTTP 2.0 / 경로 기반 라우팅** | N/A | N/A | **✅** |
| **SSL 오프로드 / Multi Zone** | ❌ | **✅** | **✅** |
| **고정 세션(Sticky Session)** | **✅** | ❌ | **✅** |
| **분산 알고리즘** | Hash, Round Robin | 3가지 모두 | 3가지 모두 |
| **CPS (초당 연결)** | Dynamic Sizing | - | 30,000 ~ 120,000 |

---

### 각 LB 특징 상세

#### Network Load Balancer (NLB) — L4
- TCP/UDP 프로토콜, 고성능 처리 특화
- **DSR (Direct Server Return)**: LB를 거치지 않고 서버가 클라이언트에 직접 응답 → 효율 극대화
- 클라이언트 실제 IP가 서버 로그에 그대로 기록됨
- Dynamic Sizing으로 트래픽에 따라 자동 성능 조절

#### Network Proxy Load Balancer — L4 Proxy
- TCP 세션 관리가 필요한 애플리케이션에 적합
- SSL 오프로드 지원 (Certificate Manager 연동)
- Classic LB와 유사한 방식

#### Application Load Balancer (ALB) — L7
- HTTP/HTTPS 헤더 분석 가능 → URL Path 기반, Host Header 기반 분기
- 가중치 기반 분기 및 Redirect 처리
- 같은 서버의 여러 포트로 분산 가능
- **CPS 보장**: 30,000 / 60,000 / 90,000 / 120,000 (설정에 따라)

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

---

## 🔑 핵심 개념 5: 포트 설정 규칙

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

## 🔑 핵심 개념 6: 모니터링

| LB 종류 | 모니터링 항목 수 |
|--------|--------------|
| Network Load Balancer | **5가지** |
| Application Load Balancer | **6가지** (Traffic Out 포함) |
| 수집 주기 | **1분, 5분, 2시간, 1일** |

---

## 📝 시험 대비 체크리스트

- [ ] Listener: 프로토콜/포트 체크 → Target Group으로 라우팅
- [ ] Round Robin: 순차 배분 / Least Connection: 최소 연결 / Source IP Hash: IP 해시
- [ ] NLB: TCP/UDP, L4, **DSR 지원**, 로깅 ❌
- [ ] Network Proxy LB: TCP/TLS, SSL 오프로드 ✅, DSR ❌
- [ ] ALB: HTTP/HTTPS, L7, URL 경로 기반, 같은 서버 여러 포트 ✅
- [ ] DSR: **NLB만** 지원, 서버 직접 응답
- [ ] LB 포트: 규칙마다 **다르게** / 서버 포트: 같아도 OK
- [ ] ALB CPS: 30,000 ~ 120,000
- [ ] 모니터링 수집 주기: 1분, 5분, 2시간, 1일

---

## 🧠 암기 핵심 문장

> "**NLB = TCP/UDP + DSR + L4 / ALB = HTTP + 경로기반 + L7**"  
> "**DSR은 NLB만, SSL 오프로드는 Proxy+ALB**"

---

*다음 챕터: `16_로드밸런서_심화.md` → Sticky Session, Idle Timeout, X-Forwarded-For*
