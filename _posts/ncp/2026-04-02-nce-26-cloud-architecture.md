---
title: "Chapter 26: 클라우드 설계의 정석 — 아키텍처 패턴과 Cloud Functions"
date: 2026-04-02 11:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, architecture, microservice, cloud-functions, serverless, iot, 3tier]
description: "NCP NCE 자격증 대비 학습 자료 - 3-tier, 모놀리식 vs 마이크로서비스, Cloud Functions, Cloud IoT Core 아키텍처 패턴"
---
> **학습 목표**: 클라우드 아키텍처 설계 패턴(3-tier, Monolithic vs Microservices)을 이해하고, NCP Cloud Functions와 Cloud IoT Core의 특징을 정확히 파악한다.

---

## 이야기: "클라우드빵집의 성장통"

작은 빵집 웹사이트였던 클라우드빵집이 폭발적으로 성장했다.

> "처음엔 PHP 하나에 DB 하나였는데... 이제 서버가 20대야."  
> "배포할 때마다 전체 시스템이 내려가는 건 좀..."  
> "이거 아키텍처를 다시 설계해야 할 것 같아."

시니어 개발자가 화이트보드를 펼쳤다.

> "**3-tier**로 나누고, 나중엔 **마이크로서비스**로 가자.  
> 그리고 간단한 이벤트 처리는 **Cloud Functions**로 분리해."

---

## 🔑 클라우드 아키텍처 접근 방식

NCP(네이버 클라우드)는 아키텍처 마이그레이션을 3단계로 구분합니다:

```
1. Lift & Shift
   - 기존 온프레미스 서버를 그대로 클라우드로 이전
   - 변경 최소화, 빠른 이전 가능
   - 클라우드 이점 활용 제한

2. Cloud-optimized
   - 클라우드 환경에 맞게 일부 최적화
   - 로드밸런서, 오토스케일링 적용

3. Cloud-native architecture
   - 클라우드를 최대한 활용하여 처음부터 설계
   - 컨테이너, 마이크로서비스, 서버리스 활용
```

---

## 🔑 소프트웨어 아키텍처 스타일

### 3-tier 아키텍처 (가장 기본!)

```
[브라우저/앱]
      ↓ HTTP 요청
┌─────────────────────┐
│ Presentation Layer  │  ← 웹 서버 (Nginx, Apache)
│   (프레젠테이션)     │    사용자 인터페이스 담당
└─────────┬───────────┘
          ↓
┌─────────────────────┐
│ Business Logic Layer│  ← 앱 서버 (Node.js, Java, Python)
│   (비즈니스 로직)    │    핵심 비즈니스 처리
└─────────┬───────────┘
          ↓
┌─────────────────────┐
│    Data Layer       │  ← DB 서버 (MySQL, Redis)
│   (데이터 접근)      │    데이터 저장 및 조회
└─────────────────────┘
```

> 💡 **가장 오래되고 가장 많이 쓰이는 아키텍처 스타일**

---

## 🔑 모놀리식 vs 마이크로서비스 (시험 핵심!)

| 항목 | **모놀리식 (Monolithic)** | **마이크로서비스 (Microservices)** |
|------|------------------------|--------------------------------|
| **구조** | 하나의 거대한 단일 애플리케이션 | 소규모 독립 서비스들의 집합 |
| **배포** | 전체를 한 번에 배포 | 서비스별 **독립적** 배포 가능 |
| **확장** | 전체 스케일 아웃 필요 | 필요한 서비스만 확장 |
| **장점** | 개발 초기 단순함 | 유연성, 독립 배포 |
| **단점** | 규모가 커질수록 복잡도 증가 | 서비스 간 통신 관리 필요 |
| **적합** | 소규모 초기 단계 | 성장하는 대규모 서비스 |

```
[모놀리식]                    [마이크로서비스]
┌──────────────────┐          ┌────────┐ ┌────────┐
│                  │          │  주문  │ │  결제  │
│  주문+결제+배달   │    →     │서비스  │ │서비스  │
│  +회원+리뷰 통합  │          └────────┘ └────────┘
│                  │          ┌────────┐ ┌────────┐
└──────────────────┘          │  배달  │ │  회원  │
  하나 바꾸면 전체 배포          │서비스  │ │서비스  │
                               └────────┘ └────────┘
                               각각 독립 배포 가능
```

---

## 🔑 Scale-up/down vs Scale-out/in

```
Scale-up/down (수직 확장)
  ├─ Scale-up: 서버 스펙을 높임 (CPU, 메모리 증가)
  └─ Scale-down: 서버 스펙을 낮춤

Scale-out/in (수평 확장)
  ├─ Scale-out: 서버 대수를 늘림 → Auto Scaling 활용
  └─ Scale-in: 서버 대수를 줄임

💡 Scale-out/in이 작동하기 위해선 N-tier Architecture를 사전에 고려해야 함!
```

### NCP Auto Scaling + Multi-AZ 패턴

```
                 [Load Balancer]
                 /             \
         [Zone A]              [Zone B]
         ┌──────┐              ┌──────┐
         │ 서버 │              │ 서버 │  ← 존 간 이중화
         │ 서버 │              │ 서버 │
         └──────┘              └──────┘
              Auto Scaling 정책으로 자동 증감
```

---

## 🔑 Cloud Functions — 서버리스 컴퓨팅

### 개념

> 서버를 직접 관리하지 않고 **코드(Action)만 실행**하는 서버리스 서비스

```
[이벤트 발생 (트리거)]
       ↓
[Cloud Functions가 코드 자동 실행]
       ↓
[결과 반환]

서버 없이 코드만! 실행된 만큼만 비용 청구
```

### 용어 정리 (시험 포인트!)

| 용어 | 설명 |
|------|------|
| **Action** | 실행되는 코드 함수 |
| **Trigger** | Action을 실행시키는 이벤트 |
| **Entity** | Action, Trigger를 통칭하는 용어. **이름 중복 불가** |

### 지원 언어 (시험 포인트!)

```
Node.js 6, 8
Python 3
Java 8
Swift 3
PHP 7
.Net
Go
```

### 코드 등록 방법

```
1. 콘솔에서 직접 작성
2. ZIP 파일로 압축하여 업로드 (Java는 JAR 파일)
   - 반드시 작성 가이드라인을 따라야 함
```

> 💡 **Entity 이름은 중복 불가** — Action과 Trigger 모두 포함해서!

---

## 🔑 Cloud IoT Core — IoT 기기 연결

### 개념

IoT 디바이스와 클라우드 서비스를 연결하는 완전 관리형 서비스

### 핵심 특징 (시험 포인트!)

| 항목 | 내용 |
|------|------|
| **통신 프로토콜** | **MQTT** (Message Queue Telemetry Transport) |
| **인증 방식** | **X.509 기반 인증서** + **TLS** 통신 |
| **처리 방식** | 실시간 MQTT 메시지 분석 및 처리 |

```
IoT 디바이스 (센서, 스마트기기)
       │  MQTT 프로토콜
       │  X.509 인증서로 인증 + TLS 암호화
       ▼
  Cloud IoT Core
       │  트리거(조건) 확인
       │  조건 일치 시 액션(동작) 수행
       ▼
  NCP 서비스 연동 (함수 실행, 알림 발송 등)
```

### MQTT 프로토콜 특징

> **경량형 메시지 프로토콜** — 성능 자원이 부족한 IoT 기기에서도 동작

- Publish/Subscribe 패턴
- 낮은 대역폭 환경에서도 동작
- IoT 분야에서 가장 널리 사용

---

## 🧠 아키텍처 패턴 총정리

| 패턴 | 특징 | NCP 서비스 |
|------|------|-----------|
| **3-tier** | Presentation / Business / Data 계층 분리 | Server + DB |
| **모놀리식** | 단일 앱, 초기 단순 | 일반 Server |
| **마이크로서비스** | 독립 배포 가능한 소규모 서비스 | NKS + Load Balancer |
| **서버리스** | 코드만 실행, 서버 관리 불필요 | **Cloud Functions** |
| **N-tier + Auto Scale** | 수평 확장 + 존 이중화 | Auto Scaling + Multi-AZ |
| **IoT** | 디바이스 연결, MQTT 통신 | **Cloud IoT Core** |

---

## 클라우드 설계 원칙 (고가용성 아키텍처)

```
✅ 좋은 클라우드 아키텍처 체크리스트

1. N-tier 분리        → 프레젠테이션/로직/데이터 계층 분리
2. 수평 확장 가능      → Auto Scaling 정책 적용
3. 존 이중화          → Multiple Availability Zones 활용
4. 단일 장애점 제거    → Load Balancer + 다중 서버
5. 부하 테스트        → 사전에 Auto Scaling 임계값 결정
6. 느슨한 결합        → 마이크로서비스, 이벤트 드리븐 설계
```

---

## 시험 직전 체크리스트

- [ ] 3-tier: Presentation / Business Logic / Data 3계층
- [ ] 모놀리식: 단일 애플리케이션, Scale-out 시 전체 확장
- [ ] 마이크로서비스: 독립 배포 가능, 서비스별 확장
- [ ] Scale-out/in이 작동하려면 **N-tier Architecture** 사전 고려
- [ ] Cloud Functions Action 지원 언어: Node.js 6/8, Python 3, Java 8, Swift 3, PHP 7, .Net, Go
- [ ] Entity = Action + Trigger 통칭, **이름 중복 불가**
- [ ] Cloud Functions 코드: 콘솔 직접 작성 또는 **ZIP(Java는 JAR)** 업로드
- [ ] Cloud IoT Core 프로토콜: **MQTT**
- [ ] Cloud IoT Core 인증: **X.509 인증서** + **TLS** 통신
- [ ] 클라우드 이전 3단계: Lift&Shift → Cloud-optimized → Cloud-native

---

**참고 문서**:
- [NCP 아키텍처 설계 가이드](https://guide.ncloud-docs.com)
- [Cloud Functions 개요](https://guide.ncloud-docs.com/docs/cloudfunctions-overview)
- [Cloud IoT Core 개요](https://guide.ncloud-docs.com/docs/iotcore-overview)
