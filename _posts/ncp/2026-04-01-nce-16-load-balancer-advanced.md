---
title: "Chapter 16: 로드밸런서 고급 기능 — Sticky Session & 심화"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, load-balancer, sticky-session, idle-timeout, x-forwarded-for]
description: "NCP NCE 자격증 대비 학습 자료 - Sticky Session, Idle Timeout, 헤더 처리 등 로드밸런서 심화"
---
> **학습 목표**: Sticky Session, Idle Timeout, X-Forwarded-Proto/For, ALB L7 고급 기능을 완벽히 이해한다.

---

## 이야기: 장바구니가 자꾸 사라져요!

클라우드빵집에 로드밸런서를 달았다.
그런데 고객 민원이 폭발했다.

> "장바구니에 담아놓은 빵이 자꾸 사라져요!"

재현이 원인을 찾았다.

> "Round Robin으로 요청을 분산하니까, 다음 요청이 다른 서버로 가서
> 세션(로그인 정보, 장바구니)이 없는 거야.
> Sticky Session을 켜야 해."

---

## 🔑 핵심 개념 1: 고정 세션 (Sticky Session)

**Sticky Session** = 동일 사용자의 요청을 **항상 같은 서버**로 보내는 기능.

```
[일반 Round Robin]
유저 A → 요청1: 서버1 (장바구니 저장)
유저 A → 요청2: 서버2 (장바구니 없음! 😱)
유저 A → 요청3: 서버3 (장바구니 없음! 😱)

[Sticky Session ON]
유저 A → 요청1: 서버1 (장바구니 저장)
유저 A → 요청2: 서버1 (항상 같은 서버 ✅)
유저 A → 요청3: 서버1 (항상 같은 서버 ✅)
```

### Sticky Session 사용 목적

| 목적 | 설명 |
|------|------|
| **로그인/세션 유지** | 로그인 정보가 특정 서버에 저장된 경우 |
| **장바구니 데이터 유지** | 서버 로컬 세션에 저장된 경우 |
| **서버 캐시 HIT 증가** | 동일 사용자 데이터를 해당 서버가 캐시 보유 → 빠른 응답 |
| **개인화 추천** | 유저 프로파일이 특정 서버에 로드된 경우 |

### Sticky Session 지원 프로토콜

```
지원: TCP ✅ / UDP ✅ / HTTP ✅ / HTTPS ✅
미지원: Network Proxy LB (TLS) ❌
```

> ⚠️ **NLB**: Sticky Session ✅ / **Network Proxy LB**: ❌ / **ALB**: ✅

---

## 🔑 핵심 개념 2: 유휴 제한 시간 (Idle Timeout)

**Idle Timeout** = LB가 클라이언트-서버 연결을 유지하는 최대 시간.  
지정 시간 동안 요청이 없으면 연결을 **자동 종료**.

```
[Idle Timeout 동작]

클라이언트 ──요청──→ LB ──→ 서버
              
              [Idle Timeout: 60초 설정]
              
              60초 동안 아무 요청 없음
                    ↓
              연결 자동 종료 (TCP RST 발송)
```

### Idle Timeout 목적

- 불필요한 연결을 줄여 **서버 리소스 절약**
- 최대 동시 접속 가능 수 증가

### Idle Timeout 연장이 필요한 상황

| 상황 | 이유 |
|------|------|
| 실시간 스트리밍 (WebSocket, SSE) | 연결 유지하며 지속적으로 데이터 수신 |
| 대용량 멀티파트 파일 업로드 | 파일 전송 시간이 길어질 수 있음 |
| 실행이 오래 걸리는 API 요청 | 배치 처리, 복잡한 연산 등 |

---

## 🔑 핵심 개념 3: SSL/HTTPS 처리와 헤더

### SSL 오프로드 (Offload)

```
[SSL 오프로드 흐름]

클라이언트 ──HTTPS(443)──→ LB ──HTTP(80)──→ 서버
          (암호화)      (복호화)  (평문)
```

- LB에서 SSL 복호화 처리 → 서버는 평문 HTTP만 처리
- 서버의 CPU 부하 감소

### X-Forwarded-Proto 헤더

> ⚠️ **시험 포인트**  
> SSL 오프로드 시, 서버는 요청이 HTTP인지 HTTPS인지 알 수 없다!  
> → LB가 **X-Forwarded-Proto** 헤더로 원본 프로토콜 정보 전달

```
[HTTP → HTTPS 리다이렉트 처리]

클라이언트 ──HTTPS──→ LB ──HTTP──→ 서버
                          ↑
              X-Forwarded-Proto: https 헤더 추가
              
서버에서 헤더 확인:
if (X-Forwarded-Proto == 'http') → HTTPS로 리다이렉트
```

### X-Forwarded-For 헤더

```
[프록시/LB 환경에서 실제 클라이언트 IP 확인]

실제 클라이언트 IP: 203.0.113.1
          ↓
         LB (10.0.1.1)
          ↓ X-Forwarded-For: 203.0.113.1 추가
         서버
```

> **HTTP_X_FORWARDED_FOR** = 프록시 환경에서 클라이언트 실제 IP를 담는 헤더

---

## 🔑 핵심 개념 4: ALB L7 고급 기능

Application LB는 HTTP 헤더를 분석해서 다양한 고급 분기 처리 가능.

| 기능 | 설명 | 예시 |
|------|------|------|
| **Host Header 기반 분기** | 도메인 이름에 따라 다른 서버 그룹으로 | bakery.com → 서버A / api.bakery.com → 서버B |
| **URL Path Pattern 기반** | URL 경로에 따라 분기 | /api/* → API 서버 / /admin/* → 관리 서버 |
| **가중치 기반 분기** | 서버별 가중치로 트래픽 비율 조절 | 서버A 80%, 서버B 20% |
| **Redirection 응답** | HTTP → HTTPS 자동 리다이렉트 | 80포트 요청 → 443으로 리다이렉트 |

```yaml
# ALB 라우팅 규칙 예시
규칙1: Host = api.bakery.com → Target Group A (API 서버)
규칙2: Path = /admin/* → Target Group B (관리 서버)
규칙3: 기본 → Target Group C (일반 웹 서버)
```

---

## 📝 시험 대비 체크리스트

- [ ] Sticky Session: 동일 사용자 → 항상 같은 서버
- [ ] Sticky Session 목적: 로그인/장바구니/캐시 HIT 유지
- [ ] Sticky Session 지원: **NLB ✅, Network Proxy LB ❌, ALB ✅**
- [ ] Idle Timeout: 지정 시간 요청 없으면 연결 종료
- [ ] Idle Timeout 연장 필요: WebSocket, 대용량 업로드, 긴 API
- [ ] SSL 오프로드: LB에서 복호화, 서버는 HTTP 처리
- [ ] **X-Forwarded-Proto**: 원본 프로토콜(http/https) 식별
- [ ] **HTTP_X_FORWARDED_FOR**: 실제 클라이언트 IP
- [ ] ALB L7: Host Header, URL Path, 가중치, Redirect

---

## 🧠 암기 핵심 문장

> "**Sticky Session = 같은 서버 고정, NLB/ALB 지원, Proxy LB 미지원**"  
> "**X-Forwarded-Proto = 프로토콜 식별 / X-Forwarded-For = 실제 IP**"

---

*다음 챕터: `17_DNS_CDN.md` → 도메인과 콘텐츠 배송 네트워크*
