---
title: "Chapter 26: 클라우드 설계의 정석 — 아키텍처 패턴과 Cloud Functions"
date: 2026-04-02 11:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, architecture, microservice, cloud-functions, serverless, iot, 3tier]
description: "NCP NCE 자격증 대비 학습 자료 - 3-tier, 모놀리식 vs 마이크로서비스, Cloud Functions, Cloud IoT Core 아키텍처 패턴"
---
> **학습 목표**: 클라우드 아키텍처 설계 패턴(3-tier, Monolithic vs Microservices)을 이해하고, NCP Cloud Functions와 Cloud IoT Core의 특징을 정확히 파악한다. 아키텍처 설계 원칙(Availability, Scalability, Elasticity, Automated, Security)을 바탕으로 실제 NCP 서비스 조합으로 아키텍처를 설계하는 방법을 익힌다.

---

## 이야기: "클라우드빵집의 성장통"

작은 빵집 웹사이트였던 클라우드빵집이 폭발적으로 성장했다.

> "처음엔 PHP 하나에 DB 하나였는데... 이제 서버가 20대야."  
> "배포할 때마다 전체 시스템이 내려가는 건 좀..."  
> "이거 아키텍처를 다시 설계해야 할 것 같아."

시니어 개발자가 화이트보드를 펼쳤다.

> "**3-tier**로 나누고, 나중엔 **마이크로서비스**로 가자.  
> 그리고 간단한 이벤트 처리는 **Cloud Functions**로 분리해."

처음엔 단순한 LAMP 스택 서버 두 대였다. 그게 이제 로드밸런서, AutoScaling, CloudDB, CDN, MQTT를 연결한 복잡한 구조로 진화해야 했다. 아키텍처란 결국 서비스 성장에 맞춰 지속적으로 진화하는 설계도인 셈이다.

---

## 🔑 소프트웨어 아키텍처란 무엇인가

소프트웨어 아키텍처는 **건축의 양식(Architectural Style)** 개념을 IT에 적용한 것이다. 과거에는 EA(Enterprise Architecture)가 유행했으며, EA는 다음 4개 영역으로 구성된다:

```
Enterprise Architecture
├── Business Architecture   (비즈니스 구조)
├── Application Architecture (애플리케이션 구조)
├── Data Architecture       (데이터 구조)
└── Technical Architecture  (기술 인프라 구조)
```

소프트웨어 아키텍처 스타일과 패턴은 매우 다양하다. 대표적인 것들:

| 스타일 | 설명 |
|--------|------|
| **Client-Server** | 클라이언트와 서버 역할 분리 |
| **N-Tier / 3-Tier** | 계층 분리 (Presentation / Logic / Data) |
| **Layered Architecture** | 계층형 구조 |
| **Domain Driven Design** | 도메인 중심 설계 |
| **Component-Based** | 컴포넌트 단위 설계 |
| **Service-Oriented (SOA)** | 서비스 지향 아키텍처 |
| **Message Bus** | 메시지 버스를 통한 통신 |
| **Object-oriented** | 객체지향 설계 |

---

## 🔑 소프트웨어 아키텍처의 진화

```
[Monolithic]          [Service Oriented]       [Microservice]
┌──────────┐          ┌────┐  ┌────┐           ┌────┐ ┌────┐ ┌────┐
│    UI    │          │    │  │    │           │MICRO│ │MICRO│ │MICRO│
│ Business │   →→→    │Svc │  │Svc │   →→→    │SVC │ │SVC │ │SVC │
│  Logic   │          │    │  │    │           └────┘ └────┘ └────┘
│ Datalayer│          └────┘  └────┘            각각 독립 DB 보유
└──────────┘         공통 UI / 공통 DB          완전한 독립성
```

### 모놀리식 (Monolithic) 아키텍처

- **정의**: 모든 비즈니스 기능이 하나의 코드베이스에 결합된 단일 컴퓨팅 환경
- "모놀리스"는 거대하고 빙하 같은 것을 의미
- 변경 시 코드베이스 전체에 접근하여 전체 스택을 업데이트해야 함
- 업데이트에 제한이 많고 시간이 오래 걸림
- 소규모 초기 서비스에 적합 ("All in one / Full Stack on single instance")

### SOA (Service Oriented Architecture)

- 기존 애플리케이션의 기능들을 **비즈니스 단위**로 묶어 **표준화된 호출 인터페이스**를 통해 서비스 컴포넌트로 재조합
- 서비스들을 **Orchestration** 하여 업무 기능 구현
- 기업의 전체 업무를 하나의 거대한 SOA 시스템으로 구성

### 마이크로서비스 (Microservice) 아키텍처

- **정의**: 서비스를 비즈니스 경계에 맞게 세분화하고, 서비스 간 통신은 네트워크 호출을 통해 진행하여 확장 가능하고 회복적이며 유연한 애플리케이션을 구성하는 것
- 각 서비스는 독립된 DB를 보유
- API Gateway를 통해 모바일/웹 클라이언트와 연결
- 공통 기능은 추상화하여 재사용

```
[마이크로서비스 구조]

모바일/웹
    │
    ▼
API Gateway (공통 기능 추상화, 서비스 중개)
    │
    ├─── API → Service1 → DB1
    ├─── API → Service2 → DB2
    └─── API → Service3 → DB3

서비스 간 통신: Remote 통신 (네트워크 호출)
```

---

## 🔑 3-tier 아키텍처 (가장 기본!)

```
[브라우저/앱]
      ↓ HTTP 요청
┌─────────────────────┐
│ Presentation Layer  │  ← 웹 서버 (Nginx, Apache)
│   (프레젠테이션)     │    사용자 인터페이스 담당
│                     │    브라우저에서 렌더링되는 정적 웹 콘텐츠 제공
└─────────┬───────────┘
          ↓
┌─────────────────────┐
│ Business Logic Layer│  ← 앱 서버 (Tomcat, Spring, Node.js)
│   (비즈니스 로직)    │    동적 웹 애플리케이션 콘텐츠 생성
│                     │    명령 처리, 논리적 결정, 연산 수행
└─────────┬───────────┘
          ↓
┌─────────────────────┐
│    Data Layer       │  ← DB 서버 (MySQL, Hadoop, Redis)
│   (데이터 접근)      │    서비스에 필요한 데이터 저장·관리
│                     │    데이터 접근(Data Access) 제공
└─────────────────────┘
```

> **가장 오래되고 가장 많이 쓰이는 아키텍처 스타일**

각 계층은 **독립된 모듈**로 개발하고 유지하며, 일반적으로 각각 다른 플랫폼에서 구동된다.

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

## 🔑 아키텍처 설계 원칙 (Design Principal)

NCP에서 강조하는 클라우드 아키텍처 설계의 5가지 핵심 원칙:

### 1. Availability (가용성)

- 서비스의 가용성을 고려
- **인프라 레벨 + 시스템 레벨 + 애플리케이션 레벨** 가용성 모두 고려
- 기법: N+1, Load Balancing, Active-Standby, Master-Slave, Clustering, Distributed

### 2. Scalability (확장성)

- 시스템의 **모든 컴포넌트**가 확장 가능하도록 설계 (데이터베이스와 스토리지 포함)
- 테스트 시스템이 프로덕션 시스템과 같은 구성이 되는 것도 고려
- 기법: Scale-out vs Scale-up, Linear Scalability, Stateless, Distributed

### 3. Elasticity (탄력성)

- 필요한 용량을 미리 계산하지 말고 **탄력적으로 운영**될 수 있도록 설계
- 과도한 용량 산정(낭비)이나 부족한 용량(성능 문제) 모두 방지
- 기법: On Demand, Auto Scaling, Self-Provisioning

### 4. Automated (자동화)

- 가능한 운영자의 개입 없이 **워크로드/트래픽 변화에 자동 반응**하도록 설계
- 기법: Auto Scaling & Resizing Web/DB/Cluster, Auto Build & Deploy, Monitoring and Alarm

### 5. Security (보안)

- 최소한의 접근, 보안 기능들의 활용

---

## 🔑 클라우드 아키텍처 접근 방식

NCP(네이버 클라우드)는 아키텍처 마이그레이션을 3단계로 구분합니다:

```
1. Lift & Shift
   - 기존 온프레미스 서버를 그대로 클라우드로 이전
   - 데이터 이전이 없거나 변환이 필요 없는 경우
   - 클라우드를 시작하기에 가장 좋은 전략
   - 기본 서비스(Compute/Storage/Network) 이용

2. Cloud-optimized
   - 클라우드 환경에 맞게 일부 최적화
   - 운영 비용 절감, 성능 향상, 서비스 운영 효율화
   - 로드밸런서, 오토스케일링 적용

3. Cloud-native architecture
   - 클라우드를 100% 사용해서 효과를 내는 개념 하의 아키텍처
   - NCP 상품 전체를 사용하여 구성 (보안 관제, 확장성 등)
   - 컨테이너, 마이크로서비스, 서버리스 활용
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
         [Zone A: KR-1]        [Zone B: KR-2]
         ┌──────────┐          ┌──────────┐
         │ LB Subnet│          │ LB Subnet│
         ├──────────┤          ├──────────┤
         │Public Sub│          │Public Sub│
         │WebServer │          │WebServer │  ← AutoScaling
         ├──────────┤          ├──────────┤
         │Private   │          │Private   │
         │Master DB │          │Standby DB│  ← 존 간 이중화
         └──────────┘          └──────────┘
              Auto Scaling 정책으로 자동 증감
```

---

## 🔑 전통 방식 vs 서버리스 — 이벤트 처리 비교

### 전통적인 방식: 서버 상시 대기

```
[서버 항상 실행 중]
  9AM ──────────────────── 12AM ────────────── 9PM
  서버 계속 떠 있음 (사용량 없어도 비용 발생)
  이벤트(request)를 대기하며 유휴 상태 지속
```

### 서버리스 방식: 이벤트 발생 시에만 자원 할당

```
  9AM                    12AM               9PM
  |        10ms    |  30ms 50ms 90ms  |
  이벤트 없으면 비용 없음
  이벤트 발생 시에만 자원 할당 및 요금 지불
```

서버리스는 **트래픽이 불규칙**하거나 **간헐적으로 발생**하는 작업에 특히 유리하다.

---

## 🔑 Cloud Functions — 서버리스 컴퓨팅

### 개념

> 이벤트에 기반한 액션을 수행하는 서버리스 서비스. 사용자의 인프라 관리 부담 없이 원하는 로직을 분산된 클라우드 환경에서 동작하고 결과를 반환합니다.

```
[이벤트 발생 (트리거)]
       ↓
[Cloud Functions가 코드 자동 실행]
       ↓
[결과 반환]

서버 없이 코드만! 실행된 만큼만 비용 청구
```

### 현재 연동 가능한 NCP 서비스

```
Cloud Functions ↔ API Gateway
Cloud Functions ↔ Database
Cloud Functions ↔ Object Storage
Cloud Functions ↔ Cloud Log Analytics
Cloud Functions ↔ SENS (SMS/푸시 알림)
```

### 구성 요소 상세 (시험 포인트!)

| 구성 요소 | 설명 |
|-----------|------|
| **Action (액션)** | 하나의 특정 작업을 수행하는 **상태가 없는(stateless) 코드 조각**. JavaScript, Swift, Java, Python, PHP 등으로 작성. **소스코드 최대 크기: 48MB** |
| **Trigger (트리거)** | 연동 가능한 클라우드 서비스나 외부 서비스에서 이벤트를 받아 와 "액션"을 실행할 수 있는 **이벤트 전달 객체**. 이벤트 발생 시 1개 이상의 액션을 **병렬로 실행** |
| **Web Action (웹 액션)** | 웹 서비스를 쉽게 제공할 수 있는 액션. 웹 기반 응용 프로그램을 만드는 데 사용. **인증키 없이 익명으로 액세스**하는 백엔드 로직 구현 가능. 인증/OAuth 기능이 필요할 경우 액션 내에서 직접 구현 |
| **Package (패키지)** | 액션과 피드를 공유하는 단위. 연관된 액션들과 피드들을 하나의 단위로 관리하고 다른 사용자와 공유 가능. Cloud Functions에서 미리 유용한 공유 패키지들을 제공 |

### 트리거 예시

```
- 주소가 업데이트될 때
- 문서가 웹사이트에 업로드될 때
- 이메일을 전송할 때
- Object Storage에 파일이 저장될 때
- 특정 시간 스케줄에 따라
```

### 용어 정리 (시험 포인트!)

| 용어 | 설명 |
|------|------|
| **Action** | 실행되는 코드 함수 (stateless) |
| **Trigger** | Action을 실행시키는 이벤트 전달 객체 |
| **Rule** | Trigger와 Action을 연결하는 규칙 |
| **Package** | 액션과 피드를 공유하는 단위 |
| **Entity** | Action, Trigger를 통칭하는 용어. **이름 중복 불가** |

### 지원 언어 (시험 포인트!)

```
Node.js 6, 8
Python 3
Java 8
Swift 3
PHP 7
.Net (C#)
Go
```

### 코드 등록 방법

```
1. 콘솔에서 직접 작성
2. ZIP 파일로 압축하여 업로드 (Java는 JAR 파일)
   - 반드시 작성 가이드라인을 따라야 함
```

> **Entity 이름은 중복 불가** — Action과 Trigger 모두 포함해서!

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
| **트리거/액션** | 사용자 정의 조건(트리거) 검사 → 일치 시 동작(액션) 수행 |

```
IoT 디바이스 (센서, 스마트기기, AI 스피커, Arduino 등)
       │  MQTT 프로토콜
       │  X.509 인증서로 인증 + TLS 암호화
       ▼
  Cloud IoT Core (Cloud Gateway / IoT Hub)
       │  트리거(조건) 확인
       │  조건 일치 시 액션(동작) 수행
       ├── Rule Engine → Stream Processing → Kafka Topic
       ├── Cloud Functions (데이터 변환)
       ├── Object Storage (Cold Data)
       └── TensorFlow / ML (머신러닝 분석)
```

### MQTT 프로토콜 특징

> **경량형 메시지 프로토콜** — 성능 자원이 부족한 IoT 기기에서도 동작

- Publish/Subscribe 패턴
- 낮은 대역폭 환경에서도 동작
- IoT 분야에서 가장 널리 사용

### IoT 분석 플랫폼 전체 구조

```
IoT Devices (센서, AI 스피커, Arduino)
     │ MQTT
     ▼
Cloud Gateway (IoT Hub)
     │
     ├─ Rule Engine ─→ Kafka Topic ─→ Stream Applications
     │                               ─→ Alarm (SMS, Email, CRM)
     │
     ├─ Cloud Functions ─→ Data Transformation ─→ Warm Data (NoSQL DB)
     │
     └─ Object Storage ─→ Cold Data
                       ─→ TensorFlow (Machine Learning)
                       ─→ UI / Reporting Tools
```

---

## 🔑 NCP 아키텍처 케이스 스터디

### Case 1: 기본 웹 서비스 아키텍처 설계

**AS-IS (기존)**: 전통적인 LAMP(Linux-Apache-MySQL-PHP) 스택, 서버 2대 이중화

**TO-BE 설계 단계:**

**1단계 — 기본 분리**
```
Load Balancer (슬랙-380758.ncloudlb.com)
     │
     ├─ Web Server x4 (10.33.40.1 ~ 10.33.40.4)
     │
     ├─ MySQL Master DB
     └─ MySQL Slave DB (블록 스토리지 추가로 용량 확장)
```

**2단계 — Auto Scaling 도입**
```
웹 서버는 AutoScaling 기능을 이용해 사용자 트래픽에 따라
서버 수가 자동으로 증가/감소

- 사용량 증가 → 웹 서버 수 자동 증가
- 사용량 감소 → 웹 서버 수 자시 감소
```

**3단계 — 운영 편의성 강화**
```
CloudDB for MySQL 도입 이유:
- 손쉬운 설치, 자동 Fail-over
- 디스크 볼륨 자동 증가
- 백업 자동화
- 읽기 부하 분산 (Read Replica)
- 모니터링 및 알람 자동화

Slave DB를 추가하고 Private Load Balancer로 바인딩하여 트래픽 분산
```

**4단계 — 파일/미디어 처리 + CDN**
```
CDN + Global CDN
      │
      ▼
Public Load Balancer
      │
    [ACG]
Auto Scaling (Web Servers)
      │
  [ACG] Private Load Balancer
      │
CloudDB for MySQL (Master/Standby/Read Replica)
      │
NAS (서버 간 공유 데이터) + Object Storage (파일 서버)

일반적인 웹 서비스 아키텍처:
CDN → LB → Web → DB → Cloud DB / Storage
```

**5단계 — 모니터링 + 보안**
```
- Cloud Insight (모니터링)
- Cloud Log Analytics (로그 분석)
- SSL VPN (관리자 서버 직접 접속용)
- Web Security Checker (웹 취약점 검사)
```

### Case 2: Multi-Zone 가용성 확보

```
[KR-1 존]                    [KR-2 존]
LB Subnet                    LB Subnet
     │                            │
Public Subnet                Public Subnet
  AutoScaling WebServer       AutoScaling WebServer
Private Subnet               Private Subnet
  Master DB              ←→  Standby DB
```

### Case 3: 서버리스 웹 아키텍처

정적 콘텐츠는 Object Storage + CDN으로 제공하고, 동적 처리는 Cloud Functions로 처리하는 구조:

```
[Static Web Content: HTML, CSS, JS, Media]
CDN → Object Storage

[Dynamic Web Application]
Users → Website → Authentication

[API 요청]
(Ajax) GET/POST
  → API Gateway (인증, 접근 제어)
  → Cloud Functions (Business Logic)
  → Cloud DB for MySQL / Redis

[Batch Job / 이메일 발송]
  → Cloud Functions → Object Storage
  → Outbound Email Cloud Functions
```

### Case 4: CLOVA를 활용한 음성인식 계좌이체

```
1. 음성 입력: "아들에게 십만원 이체해줘"
   ↓
2. CLOVA Speech Recognition → 텍스트 변환
   ↓
3. CLOVA Chatbot → 사용자 의도 파악
   ↓
4. Banking System Back-end (Load Balancer → Web/WAS) → 요구사항 처리
```

### Case 5: OCR + Papago 번역 시스템

```
[이미지 번역]
이미지 전달 → CLOVA OCR → Text 추출
                             ↓
                         Papago Translation → 번역된 Text → Application

[음성 번역]
음성 전달 → CLOVA Speech Recognition → Text 변환
                                          ↓
                                      Papago Translation → 번역된 Text
```

### Case 6: OCR을 이용한 이미지 텍스트 추출 후 데이터 저장

```
1. Server에서 OCR Invoke URL로 분석 대상 이미지 전달
2. API Gateway에서 인증 처리 후, OCR로 전달
3. OCR에서 이미지 분석 후, 텍스트 리턴
   예시 응답:
   "fields": [
     {"inferText": "하늘로", "inferConfidence": 0.9998},
     {"inferText": "돌아가리", "inferConfidence": 0.9999},
     ...
   ]
4. Server에서 리턴 받은 분석 텍스트를 Cloud DB에 저장
```

### Case 7: CI/CD Pipeline

```
[CI - Continuous Integration]
Admin + SSL VPN
  → SourceCommit (App Code 커밋 감지)
  → SourceBuild (빌드, Code Scan, 파일 저장)
  → Container Registry (이미지 push)
  → Object Storage (빌드 파일 저장)
  → FileSafer / CloudInsight 연동

[CD - Continuous Deployment]
Source Pipeline (commit 감지 후 Deploy)
  → SourceCommit (deploy code, k8s manifest update)
  → Source Pipeline (CD)
  → Kubernetes (deployment 업데이트 및 배포 전략에 따라 Pod 재생성)

[모니터링]
Grafana + Loki → Logging + Monitoring
```

### Case 8: TensorFlow 머신러닝 파이프라인

```
[학습 데이터 준비]
Raw Data Source → Raw Data Store
  → Data Cleansing & Transformation
  → Parquet Formatted Data Store
  → Extract Feature → Feature Repository

[모델 학습/평가]
Object Storage → Training / Evaluation → MODEL

[서빙]
New Input Data → ML Algorithm → Prediction → Query / Dashboard
                                           → Ranking / Prediction
```

---

## 🧠 아키텍처 패턴 총정리

| 패턴 | 특징 | NCP 서비스 |
|------|------|-----------|
| **3-tier** | Presentation / Business / Data 계층 분리 | Server + DB |
| **모놀리식** | 단일 앱, 초기 단순 | 일반 Server |
| **SOA** | 서비스 지향, 표준 인터페이스 | API Gateway + Server |
| **마이크로서비스** | 독립 배포 가능한 소규모 서비스 | NKS + Load Balancer |
| **서버리스** | 코드만 실행, 서버 관리 불필요 | **Cloud Functions** |
| **N-tier + Auto Scale** | 수평 확장 + 존 이중화 | Auto Scaling + Multi-AZ |
| **IoT** | 디바이스 연결, MQTT 통신 | **Cloud IoT Core** |
| **AI/ML** | 음성인식, OCR, 번역, ML | CLOVA + Papago + TensorFlow |

---

## 클라우드 설계 원칙 (고가용성 아키텍처)

```
✅ 좋은 클라우드 아키텍처 체크리스트

1. N-tier 분리          → 프레젠테이션/로직/데이터 계층 분리
2. 수평 확장 가능        → Auto Scaling 정책 적용
3. 존 이중화            → Multiple Availability Zones (KR-1, KR-2) 활용
4. 단일 장애점 제거      → Load Balancer + 다중 서버, SPoF 제거
5. DB 이중화            → CloudDB Master-Slave, Auto Fail-over
6. 부하 테스트           → nGrinder 등으로 사전 Auto Scaling 임계값 결정
7. 느슨한 결합           → 마이크로서비스, 이벤트 드리븐 설계
8. 모니터링 자동화        → CloudInsight + CloudLogAnalytics 연동
9. 보안 레이어           → ACG + SSL VPN + Web Security Checker
10. CDN 활용            → 이미지/동영상 등 정적 파일 전송 최적화
```

---

## 시험 직전 체크리스트

- [ ] 3-tier: Presentation / Business Logic / Data 3계층 (각각 독립된 모듈, 다른 플랫폼 구동)
- [ ] 모놀리식: 단일 코드베이스, Scale-out 시 전체 확장, 변경 시 전체 스택 업데이트
- [ ] SOA: 비즈니스 단위로 묶어 표준화된 호출 인터페이스, Orchestration
- [ ] 마이크로서비스: 비즈니스 경계로 세분화, 네트워크 호출 통신, 독립 DB, API Gateway
- [ ] Scale-out/in이 작동하려면 **N-tier Architecture** 사전 고려
- [ ] 클라우드 이전 3단계: Lift&Shift → Cloud-optimized → Cloud-native
- [ ] 설계 원칙 5가지: **Availability, Scalability, Elasticity, Automated, Security**
- [ ] 전통방식: 서버 상시 대기, 유휴 비용 발생
- [ ] 서버리스: 이벤트 발생 시에만 자원 할당 및 요금 지불
- [ ] Cloud Functions 연동 서비스: API Gateway, Database, Object Storage, Cloud Log Analytics, **SENS**
- [ ] Cloud Functions Action 지원 언어: Node.js 6/8, Python 3, Java 8, Swift 3, PHP 7, .Net(C#), Go
- [ ] Action 소스코드 최대 크기: **48MB**
- [ ] Trigger: 이벤트 발생 시 1개 이상의 Action을 **병렬로** 실행
- [ ] Web Action: 인증키 없이 **익명으로** 액세스 가능한 백엔드 로직
- [ ] Package: 액션과 피드를 공유하는 단위, 다른 사용자와 공유 가능
- [ ] Entity = Action + Trigger 통칭, **이름 중복 불가**
- [ ] Cloud Functions 코드: 콘솔 직접 작성 또는 **ZIP(Java는 JAR)** 업로드
- [ ] Cloud IoT Core 프로토콜: **MQTT** (경량형 메시지 프로토콜)
- [ ] Cloud IoT Core 인증: **X.509 인증서** + **TLS** 통신
- [ ] Cloud IoT Core: 실시간 MQTT 메시지 분석 및 처리, 트리거/액션 수행
- [ ] 웹 서비스 아키텍처: CDN → LB → Web → DB → Cloud DB / Storage
- [ ] Multi-Zone 이중화: KR-1 + KR-2 이용

---

**참고 문서**:
- [NCP 아키텍처 설계 가이드](https://guide.ncloud-docs.com)
- [Cloud Functions 개요](https://guide.ncloud-docs.com/docs/cloudfunctions-overview)
- [Cloud IoT Core 개요](https://guide.ncloud-docs.com/docs/iotcore-overview)
