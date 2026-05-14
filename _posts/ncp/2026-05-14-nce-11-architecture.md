---
title: "[NCE] 11. 아키텍처 설계 & 케이스 스터디"
date: 2026-05-14 10:40:00 +0900
categories: [NCP, NCE]
tags: [NCP, NCE, 아키텍처, 3-Tier, Microservice, Auto Scaling, HA, MultiZone, IoT, MQTT, CI/CD]
description: "소프트웨어 아키텍처 패턴, 3-Tier 구조, 클라우드 설계 원칙, Lift-and-Shift/Cloud-native, 웹서비스/서버리스/IoT 케이스 스터디 정리"
---

## 핵심 요약

- 3-Tier: Presentation → Business Logic → Data
- 웹서비스 아키텍처 공식: CDN → LB → Web → DB → Storage
- SPoF 제거 3요소: 서버 다중화 + Cloud DB 이중화 + MultiZone
- 클라우드 접근법: Lift-and-Shift → Cloud-optimized → Cloud-native
- IoT 프로토콜: MQTT (경량형 메시지 프로토콜), X.509 인증 + TLS

## 소프트웨어 아키텍처 패턴

| 패턴 | 설명 |
|------|------|
| **Layered Architecture** | 계층별 분리 (Presentation / Business / Data) |
| **Client-Server** | 클라이언트와 서버 역할 분리 |
| **N-Tier / 3-Tier** | 다계층 구조 |
| **Service-Oriented (SOA)** | 서비스 단위로 구성 |
| **Microservice** | 서비스를 비즈니스 경계로 세분화 |
| **Message Bus** | 메시지 기반 통신 |
| **Domain Driven Design** | 도메인 중심 설계 |

## 아키텍처 진화: Monolithic → SOA → Microservice

### Monolithic (모놀리식)

- 모든 비즈니스 로직을 **하나의 코드베이스**에 결합
- 변경 시 **전체 스택을 업데이트**해야 함
- 업데이트에 제한 많고 시간이 오래 걸림

### SOA - Service Oriented Architecture

- 기능을 **서비스(컴포넌트)로 재조합**하고 표준화된 인터페이스로 호출
- 서비스들을 조합(Orchestration)하여 업무 구현
- 기업 전체 업무가 하나의 거대한 SOA 시스템 구성

### Microservice Architecture

- 서비스를 **비즈니스 경계에 맞게 세분화**, 서비스 간 **네트워크 호출**로 통신
- 독립적으로 배포 가능한 유닛
- **API Gateway**를 통해 서비스 중개 및 공통 기능 추상화

```
Monolithic        SOA            Microservice
   UI              UI             웹/모바일 앱
   BL      →       BL      →    API Gateway
   Data          services      Service1 Service2 Service3
                                  DB1      DB2      DB3
```

## 3-Tier 아키텍처

| 계층 | 역할 | 예시 |
|------|------|------|
| **Presentation Layer** | 사용자 인터페이스, 정적 웹 콘텐츠 | Apache, Nginx |
| **Business Logic Layer** | 동적 콘텐츠 생성, 비즈니스 로직 | Tomcat, Spring |
| **Data Layer** | 데이터 저장/관리 | MySQL, Hadoop |

- 각 계층은 **독립된 모듈**로 개발 및 유지

## 클라우드 설계 원칙

| 원칙 | 적용 방법 |
|------|----------|
| **Availability (가용성)** | N+1, Load Balancing, Active-Standby, Clustering |
| **Scalability (확장성)** | Scale-out, Scale-up, Stateless, Distributed |
| **Elasticity (탄력성)** | On Demand, Auto Scaling, Self-Provisioning |
| **Automated (자동화)** | Auto Scaling, Auto Build & Deploy, Monitoring |
| **Security (보안)** | ACG, WAF, SSL VPN, Security Checker |

## 클라우드 마이그레이션 접근법

| 접근법 | 설명 |
|--------|------|
| **Lift-and-Shift** | 애플리케이션 그대로 클라우드 VM에 이전. 가장 쉬운 시작 전략 |
| **Cloud-optimized** | 클라우드 전환하면서 가치를 더하는 아키텍처 구성. 운영비용 절감 |
| **Cloud-native** | 클라우드를 **100% 활용**. NCP 전체 상품 사용하여 보안/확장성 극대화 |

## 사용자 증가에 따른 아키텍처 발전

| 단계 | 규모 | 구성 |
|------|------|------|
| **단계 1** | 소규모 | 단일 VM에 웹서버 + DB + 관리도구 (All-in-one) |
| **단계 2** | 수백~수천 명 | Scale-up/out, 웹서버/DB 서버 분리 |
| **단계 3** | HA 구성 | 서버 다중화 + LB + Cloud DB 이중화 + MultiZone |

**SPoF 제거 3요소:**
1. 서버 다중화 + Load Balancing
2. Cloud DB 이중화 (Master + Standby)
3. **Multiple Availability Zones** (KR-1, KR-2) 존간 이중화

## 케이스 스터디

### 기본 웹서비스 아키텍처 공식

```
CDN → LB → Web → DB → Storage
```

**NCP 구성 예시:**
```
Internet
    ↓
CDN+ / GlobalEdge (이미지/동영상 캐싱)
    ↓
ALB (LoadBalancer)
    ↓
WebServer × 4대 (Auto Scaling)
    ↓
NAS (공유 데이터)  |  ObjectStorage (파일 서버)
    ↓
Cloud DB for MySQL (Master + Standby + Read Replica)
    ↓
모니터링: Cloud Insight + Cloud Log Analytics
관리 접속: SSL VPN  |  보안 점검: Web Security Checker
```

### MultiZone 가용성 구성

```
Internet
    ↓
ALB (LB Subnet - 존 간 공유)
    ├── [KR-1 Zone]
    │     Public Subnet: AutoScaling WebServer
    │     Private Subnet: Master DB
    └── [KR-2 Zone]
          Public Subnet: AutoScaling WebServer
          Private Subnet: Standby DB
```

### 서버리스 Web 아키텍처

```
[정적 콘텐츠]
CDN + ObjectStorage (HTML/CSS/JS/Media)

[동적 콘텐츠]
Users → API Gateway (인증/접근 제어)
              ↓
        Cloud Functions (비즈니스 로직)
              ↓
        Cloud DB for MySQL / Redis

[배치]
Cloud Functions → ObjectStorage
[이메일]
Cloud Functions → Outbound Mailer
```

### 음성인식 계좌이체 시스템

**사용 서비스**: CLOVA Speech Recognition + CLOVA Chatbot

```
"아들에게 10만원 이체해줘" (음성 입력)
    ↓
1. CLOVA Speech Recognition → 텍스트 변환
2. 텍스트 → CLOVA Chatbot → 의도 파악
3. Back-end → Banking System 처리
```

### 이미지/음성 번역 시스템

**사용 서비스**: CLOVA OCR / CLOVA Speech Recognition + Papago

```
이미지 → CLOVA OCR → 텍스트 → Papago → 번역된 텍스트
음성   → CLOVA Speech Recognition → 텍스트 → Papago → 번역된 텍스트
```

### CI/CD Pipeline 설계

```
[CI]
SourceCommit (코드 커밋)
    → SourceBuild (빌드)
    → FileSafer (코드 스캔)
    → ObjectStorage (빌드 파일) / ContainerRegistry (이미지)

[CD]
SourcePipeline → SourceDeploy
    → Staging → Production (Rolling/Blue-Green/Canary)

모니터링: Cloud Insight + Notification Manager
```

### IoT 분석 플랫폼

**핵심 프로토콜:**
- **MQTT (Message Queue Telemetry Transport)**: 성능/자원이 부족한 장비에서도 동작하는 **경량형 메시지 프로토콜**, IoT에서 널리 사용
- **Cloud IoT Core**: **X.509 기반 인증서** 사용, TLS 통신 지원

```
IoT Devices (Arduino, AI Speaker 등)
    ↓ MQTT
MQTT Broker → Rule Engine → Kafka Topic → Stream Applications
    ↓
Cloud Gateway (IoT Hub)
    ↓
Object Storage (Cold Data) → TensorFlow (ML)
Cloud Functions (변환) → NoSQL DB (Warm Data)
    ↓
UI/Reporting Tools
```

### ML 파이프라인

```
1. Training Data 준비
2. ML Algorithm으로 학습
3. 예측값 vs 실제 결과값 비교
4. 가중치 조정 후 재학습
5. 성능 검증 → 모델 완성
6. 실제 상황에 모델 사용

New Input Data → ML Algorithm → Prediction Result
```

## 시험 암기 포인트

- **3-Tier**: Presentation(Apache/Nginx) → Business Logic(Tomcat/Spring) → Data(MySQL)
- **웹서비스 공식**: CDN → LB → Web → DB → Storage
- **SPoF 제거**: 서버 다중화 + Cloud DB 이중화 + MultiZone(KR-1/KR-2)
- **Lift-and-Shift**: 그대로 이전 (가장 쉬운 시작)
- **Cloud-native**: NCP 100% 활용
- **Elasticity**: Auto Scaling + Self-Provisioning + On Demand
- **음성 계좌이체**: CLOVA Speech Recognition + CLOVA Chatbot
- **이미지/음성 번역**: CLOVA OCR/CSR + Papago Translation
- **IoT 프로토콜**: MQTT (경량형), X.509 인증 + TLS
- **CI/CD**: SourceCommit → SourceBuild → SourceDeploy (SourcePipeline으로 통합)

## 원문 반영 체크

- 반영 페이지: 343~375p
- 포함 내용: 소프트웨어 아키텍처 패턴, Monolithic/SOA/MSA 진화, 3-Tier, 클라우드 설계 원칙, 마이그레이션 접근법, 아키텍처 발전 단계, 웹서비스/서버리스/음성인식/번역/CI-CD/IoT/ML 케이스 스터디
