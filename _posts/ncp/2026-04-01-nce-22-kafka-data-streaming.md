---
title: "Chapter 22: 실시간 데이터 고속도로 — Kafka & 데이터 스트리밍"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, kafka, rabbitmq, cdss, streaming, dataflow]
description: "NCP NCE 자격증 대비 학습 자료 - Kafka, CDSS, Data Flow 기반 실시간 데이터 처리 정리"
---
> **학습 목표**: Kafka vs RabbitMQ 차이, CDSS 최소 노드 구성, Data Forest/Box Frame/Catalog/Flow 서비스 특징을 완벽히 이해한다.

---

## 이야기: 실시간 주문이 밀려온다

클라우드빵집 특가 이벤트 날.
초당 1만 건의 주문이 쏟아졌다.

> "DB에 직접 쓰면 터질 것 같아! 메시지 큐에 먼저 받아야 해!"

재현이 말했다.

> "Kafka 써. 수백만 건도 실시간으로 처리해.
> RabbitMQ랑 다르게 데이터가 삭제 안 되고 재처리도 가능해."

---

## 🔑 핵심 개념 0: 메시징 시스템 기초

**메시징 시스템**이란, 애플리케이션 간에 메시지를 교환하기 위해 사용하는 시스템이다.

여기서 **메시지**란 로그 데이터, 이벤트 메시지 등 API로 호출할 때 보내는 데이터를 의미한다.

메시징 시스템은 크게 두 가지 모델로 구분된다:

| 모델 | 설명 |
|------|------|
| **Point to Point** | 보내는 사람이 큐를 통해 메시지를 전달하면 받는 사람이 큐에서 하나씩 꺼내 읽는 방식 |
| **Pub/Sub** | Publisher(게시자)가 Topic에 메시지를 보내면, 해당 Topic을 구독한 Subscriber(구독자) 모두에게 메시지가 전송되는 방식 |

대표적인 메시징 시스템: **Kafka, RabbitMQ, ActiveMQ, AWS SQS, Java JMS**

---

## 🔑 핵심 개념 1: Kafka 기본 구조 (⭐⭐ 최빈출!)

**Apache Kafka** = **분산 메시지 스트리밍 플랫폼**. 대용량 실시간 데이터 처리.

Kafka는 원래 LinkedIn에서 여러 일반 앱·SaaS 앱·구직/채용 정보들을 한 곳에서 처리(발행/구독)할 수 있는 플랫폼으로 개발이 시작되었다. 설계 당시부터 카프카를 메시지 전달의 중앙 플랫폼으로 두고, 기업에서 필요한 모든 데이터 시스템과 연결된 파이프라인을 만드는 것을 목표로 했다.

**대용량 실시간 로그 처리에 특화**되어 설계된 메시징 시스템으로, 기존 범용 메시징 시스템 대비 TPS(초당 처리 건수)가 우수하다.

### Kafka 주요 개념

```
[Kafka 아키텍처]

Producer → [Topic (Partition 1)] → Consumer Group A
           [Topic (Partition 2)] → Consumer Group B
           [Topic (Partition 3)] → Consumer Group C
           
→ Topic: 메시지 카테고리
→ Partition: Topic을 분산 저장하는 단위
→ Broker: Kafka 서버 (여러 대로 구성)
```

| 개념 | 설명 |
|------|------|
| **Message** | 카프카에서 다루는 데이터의 최소 단위. Key와 Value 값을 가짐 |
| **Producer** | 데이터의 생산자이며 브로커에 메시지를 보내는 애플리케이션 |
| **Consumer** | 브로커에서 메시지를 취득하는 애플리케이션 |
| **Topic** | 프로듀서와 컨슈머들이 카프카로 보낸 메시지를 구분하기 위한 이름 |
| **Partition** | 병렬 처리가 가능하도록 Topic을 나눌 수 있고, 많은 양의 메시지 처리를 위해 파티션의 수를 늘릴 수 있음 |
| **Broker** | 메시지 수집/전달 역할을 하는 Kafka 서버 인스턴스 |
| **Zookeeper** | Kafka 클러스터 메타데이터 관리 (클러스터 등록/관리) |

프로듀서와 컨슈머는 특정 토픽을 지정하여 메시지를 송수신함으로써 단일 Kafka 클러스터에서 여러 종류의 메시지를 중계한다.

### Kafka의 핵심 특성

```
[Kafka 데이터 저장 방식]

일반 메시지 큐:
Producer → Queue → Consumer (읽으면 삭제!)

Kafka:
Producer → Topic (파일 시스템 영구 저장)
              ↓          ↓           ↓
          Consumer A  Consumer B  Consumer C
          (각자 독립적으로 읽기 / 데이터 유지!)
```

| 특성 | 설명 |
|------|------|
| **저장 방식** | **파일 시스템** (메모리 X) — 서비스 재시작 시 메시지 유실 감소 |
| **소비 방식** | **Pull** (Consumer가 브로커에서 직접 가져감) |
| **데이터 보존** | 소비 후에도 **삭제 안 됨** → 재처리 가능 |
| **배달 보장** | At-least-once (적어도 한 번) |

> Pull 방식이므로 Consumer는 자신의 처리 능력만큼의 메시지만 브로커로부터 가져오기 때문에 **최적의 성능**을 낼 수 있다.

---

## 🔑 핵심 개념 2: Kafka vs RabbitMQ (⭐⭐ 시험 단골 비교!)

| 항목 | **Kafka** (유튜브 스트리밍) | **RabbitMQ** (택배 시스템) |
|------|-----------|-------------|
| **모델** | **Topic** (Pub/Sub) | **Queue** (Point to Point) |
| **전달 방식** | **Pull** (Consumer가 가져감) | **Push** (서버가 밀어줌) |
| **소비 후 데이터** | **유지** (재처리 가능, 오래 저장 가능) | **삭제** (배달되면 삭제) |
| **저장 방식** | **파일 시스템** | 메모리 (일부 디스크) |
| **순서 보장** | Partition 내에서만 | Queue 내 FIFO |
| **처리량** | **대용량** (초당 수백만 개 처리 가능) | 빠르지만 대량 처리에는 약함 |
| **사용 용도** | 대량 데이터 스트리밍, 로그 처리 | 요청-응답, 비동기 처리 (마이크로서비스) |

> ⚠️ **시험 함정**: 
> - Kafka = Topic + Pull + 파일 시스템 + 삭제 안 됨
> - RabbitMQ = Queue + Push + 소비 후 삭제

```
[한 줄 비교]
Kafka = "신문 구독 (배달 온 신문은 집에 쌓임, 다시 읽기 가능)"
RabbitMQ = "줄 서기 (내가 앞으로 나가면 다음 사람이 처리)"
```

---

## 🔑 핵심 개념 3: CDSS (Cloud Data Streaming Service) (⭐⭐ 빈출!)

**CDSS** = NCP의 **관리형 Kafka 서비스**. Kafka를 직접 설치/관리하지 않고 사용.

이벤트/데이터 스트리밍 처리를 위한 Apache Kafka 클러스터 서비스로, 콘솔을 몇 번만 클릭하여 사용자가 원하는 구성을 설정하면 Apache Kafka 클러스터 및 Apache ZooKeeper를 자동으로 프로비저닝해 준다.

### CDSS 아키텍처

```
[CDSS 데이터 흐름]

Data Source → Cloud Data Streaming Service → Destination

Producer                Kafka Cluster              Consumer
Application        Broker 101: TopicA Partition0(Leader)    Spark
(Java, Python...)  Broker 102: TopicA Partition0(Replica)   Cloud Hadoop
                   Broker 103: TopicA Partition1(Leader)    fluentd
      send()             TopicA Partition1(Replica)   poll()  logstash
```

### CDSS 클러스터 관리 도구

오픈소스 **CMAK(Cluster Manager for Apache Kafka)**를 사용하여 Apache Kafka 클러스터를 관리한다. CMAK를 통해 클러스터, 토픽 등의 생성 및 변경, Consumer group 확인 등 Kafka 클러스터 관리 기능을 제공한다.

> CMAK 접속을 위해서는 **Public 도메인을 활성화**해야 한다.  
> 클러스터 관리 > CMAK 접속 도메인 설정 변경을 통해 활성화 가능.

### CDSS 노드 구성 (⭐⭐ 최빈출!)

```
[CDSS 최소 구성]

┌─────────────────┐
│   Manager Node  │  ← ZooKeeper 역할 (Kafka 클러스터 관리)
│      ×1         │
└─────────────────┘
         ↕
┌──────┐ ┌──────┐ ┌──────┐
│Broker│ │Broker│ │Broker│
│  1   │ │  2   │ │  3   │
└──────┘ └──────┘ └──────┘
  최소 3대 브로커 필수!

→ 최소 합계: 1 + 3 = 4 노드
```

| 항목 | 내용 |
|------|------|
| **최소 노드 수** | **4개** (Manager 1 + Broker 3) |
| **Broker 확장** | 증가는 가능, **감소는 불가** |
| **Broker 추가 시** | 전체 클러스터가 재시작됨 |
| **Broker 타입 변경** | 기존 브로커 노드의 서버 타입과 동일하게 생성되며 **변경 불가** |
| **한 번에 최대 추가** | 최대 10대까지 추가 가능 |
| **Manager 역할** | 클러스터 메타데이터 관리 |

> ⚠️ **시험 함정**:
> - CDSS 최소 = **4 노드** (1 manager + 3 broker)
> - Broker 수는 **줄일 수 없음** (데이터 안정성 때문)
> - Broker 추가 시 전체 클러스터 **재시작** 발생

### Lab 15: CDSS 생성 및 데이터 전송 실습 흐름

1. Cloud Data Streaming Service 생성
2. Log generator 다운로드 및 **Logstash** 연동 설정
3. 로그 생성 및 Logstash를 통해 CDSS로 전송
4. CMAK Public domain 활성화
5. CMAK에 접속하여 토픽 확인

> Logstash는 CDSS의 Consumer 측에 위치하여 데이터를 수집/전송하는 역할을 한다.

---

## 🔑 핵심 개념 4: 데이터 분석 플랫폼 서비스

### Data Forest

```
[Data Forest = Apache Hadoop 기반 빅데이터 처리 클러스터]

User1(Batch Job) ──→ DataForest ──→ Object Storage
User2(Long-Live Job)             ──→ AI Forest (GPU 딥러닝)
```

- **Apache Hadoop 기반**의 대용량 멀티테넌트 빅데이터 처리 클러스터
- YARN 애플리케이션 형태로 서비스를 실행하며, 사용자가 애플리케이션을 조합하여 빅데이터 에코시스템을 만들 수 있는 환경 제공
- 사용자별 **GPU 리소스를 동적으로 할당**받아 TensorFlow, PyTorch 등의 딥러닝 학습 수행 가능
- **컨테이너 기반**이기 때문에 온라인 상태에서 동적으로 확장이 가능하며 필요 시 빠르게 변경 가능

### Data Box Frame

```
[Data Box Frame = 데이터 분석 플랫폼]

내/외부 분석자 → DataBox Frame → 안전한 분석 환경
                  (관리자 승인)   (데이터 반출 통제)
```

- 보유하고 있는 데이터를 내/외부 분석자들이 **안전하게 분석할 수 있는 환경**을 클라우드 상에서 손쉽게 만들고 운영할 수 있는 데이터 분석 플랫폼
- **분석자** 권한: DataBox 관리자로부터 제공받은 분석 대상 데이터를 자유롭게 열람하고 분석 가능. 단, 데이터를 **반출하려는 경우 반드시 DataBox 관리자에게 타당성 심사를 받은 후 승인**을 얻어야만 진행 가능
- **DataBox 관리자** 기능: 분석 대상 데이터 관리, 분석자별 열람 통제, 분석자의 개별 데이터 반입, 분석자별 분석 결과 반출, 반출 타당성 심사, 분석자별 비용 조회 등 서비스 운영 및 데이터 관리에 필요한 부가 기능 제공

### Data Catalog

```
[Data Catalog = 메타데이터 수집 자동화 + 데이터 자산 관리]

"어디에 어떤 데이터가 있나?"
→ 데이터 스캔 자동화 → 메타데이터 주기적 수집 → 스키마 추론 → 테이블 생성
```

Data Catalog의 핵심 기능:

| 기능 | 설명 |
|------|------|
| **메타데이터 수집 자동화** | 데이터 스캔을 자동화하여 메타데이터를 주기적으로 수집. 수집한 데이터에서 **스키마를 추론**하여 데이터에 맞는 테이블 생성 |
| **다양한 데이터 소스 연동** | 커넥션 설정으로 **Cloud DB** 및 **Object Storage**에 연결하여 데이터 활용 |
| **태그를 활용한 검색** | 데이터베이스와 테이블에 태그를 추가하여 쉽게 식별 및 검색 가능 |
| **테이블 및 스키마 변경 이력 관리** | 변경된 테이블 및 스키마 버전을 관리하고 시간별 변경 내역 확인 가능 |

### Data Flow (완전 관리형 서버리스 ETL) (⭐)

```
[Data Flow = 완전 관리형 서버리스 데이터 통합 서비스]

Extract → Transform → Load

시각적 인터페이스로 코딩 없이 ETL 파이프라인 구성
→ 인프라 관리 리소스 절약 (경제적)
```

- **완전 관리형 서버리스** 데이터 통합 서비스
- 웹 서버, 데이터베이스, 파일, 애플리케이션 로그 등 여러 매체의 다양한 데이터 소스를 처리하는 파이프라인을 효과적으로 제공하는 서버리스 서비스
- 클라우드 기반의 완전 관리형 서비스로 제공되기 때문에 **인프라 관리에 필요한 리소스를 절약**할 수 있어 경제적

#### NCP 서비스 연동 (⭐ 중요!)

DataFlow는 NCP 서비스 중 다음 3가지와의 연동을 지원하며, 서비스를 **소스 데이터 노드**와 **타깃 데이터 노드**로 사용한다:

| 연동 서비스 | 역할 |
|------------|------|
| **DataCatalog** | 소스/타깃 데이터 노드 |
| **Object Storage** | 소스/타깃 데이터 노드 |
| **CloudDB for MySQL** | 소스/타깃 데이터 노드 |

#### 시각적 인터페이스

- 시각적 인터페이스를 통해 **코드 작성 없이** ETL(Extract, Transform, and Load) 작업 및 데이터 파이프라인을 쉽게 구성하고 편집 가능
- GUI 기반 드래그&드롭으로 파이프라인 설계 → 개발자 없이도 데이터 처리 자동화

---

### 데이터 플랫폼 서비스 비교 요약

| 서비스 | 핵심 키워드 | 특이점 |
|--------|-----------|--------|
| **Data Forest** | Hadoop 기반 빅데이터 클러스터 | GPU 딥러닝 지원, 컨테이너 기반 동적 확장 |
| **Data Box Frame** | 데이터 분석 플랫폼 | 반출 시 관리자 승인 필수 |
| **Data Catalog** | 메타데이터 수집 자동화, 스키마 추론 | CloudDB + Object Storage 연동 |
| **Data Flow** | 완전 관리형 서버리스 ETL | DataCatalog + Object Storage + CloudDB for MySQL 연동 |

---

## 📝 시험 대비 체크리스트

**Kafka:**
- [ ] 모델: **Topic** (RabbitMQ는 Queue)
- [ ] 전달 방식: **Pull** (RabbitMQ는 Push)
- [ ] 저장 방식: **파일 시스템** (메모리 X → 재시작 시 유실 감소)
- [ ] 소비 후: **데이터 유지** (재처리 가능)
- [ ] LinkedIn에서 개발 시작, 기존 메시징 시스템 대비 TPS 우수

**RabbitMQ vs Kafka:**
- [ ] Kafka = Topic + Pull + 파일 시스템 + 삭제 안 됨 + 대량 데이터 스트리밍
- [ ] RabbitMQ = Queue + Push + 소비 후 삭제 + 마이크로서비스 비동기 처리

**CDSS:**
- [ ] 최소 노드: **4개** (Manager 1 + Broker 3)
- [ ] Broker 수: 증가 ✅ / **감소 ❌**
- [ ] Broker 추가 시 전체 클러스터 **재시작**
- [ ] 클러스터 관리 도구: **CMAK** (Public 도메인 활성화 필요)
- [ ] Logstash 연동 지원

**데이터 플랫폼:**
- [ ] Data Forest = **Hadoop 기반** 빅데이터 클러스터, GPU 딥러닝 지원
- [ ] Data Box Frame = 데이터 분석 플랫폼, **데이터 반출 시 관리자 승인 필수**
- [ ] Data Catalog = **메타데이터 수집 자동화 + 스키마 추론**, CloudDB + Object Storage 연동
- [ ] Data Flow = **완전 관리형 서버리스 ETL**, DataCatalog + Object Storage + CloudDB for MySQL 연동

---

## 🧠 암기 핵심 문장

> "**Kafka = Topic + Pull + 파일저장 + 재처리 가능**"  
> "**RabbitMQ = Queue + Push + 소비 후 삭제**"  
> "**CDSS 최소 = 4노드 (Manager 1 + Broker 3), Broker 감소 불가, 추가 시 전체 재시작**"  
> "**Data Catalog = 메타데이터 자동 수집 + 스키마 추론**"  
> "**Data Flow = 서버리스 ETL + DataCatalog/Object Storage/CloudDB for MySQL 연동**"

---

*다음 챕터: `23_보안_웹취약점_보안상품.md` → 웹 보안, KMS, Private CA, WAF*
