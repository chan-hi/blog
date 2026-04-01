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

## 🔑 핵심 개념 1: Kafka 기본 구조 (⭐⭐ 최빈출!)

**Apache Kafka** = **분산 메시지 스트리밍 플랫폼**. 대용량 실시간 데이터 처리.

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
| **Producer** | 메시지를 Kafka에 발행하는 측 |
| **Consumer** | 메시지를 Kafka에서 가져오는 측 |
| **Topic** | 메시지를 분류하는 카테고리 |
| **Partition** | Topic을 분산 저장하는 단위 |
| **Broker** | Kafka 서버 인스턴스 |

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
| **저장 방식** | **파일 시스템** (메모리 X) |
| **소비 방식** | **Pull** (Consumer가 가져감) |
| **데이터 보존** | 소비 후에도 **삭제 안 됨** → 재처리 가능 |
| **배달 보장** | At-least-once (적어도 한 번) |

---

## 🔑 핵심 개념 2: Kafka vs RabbitMQ (⭐⭐ 시험 단골 비교!)

| 항목 | **Kafka** | **RabbitMQ** |
|------|-----------|-------------|
| **모델** | **Topic** | **Queue** |
| **전달 방식** | **Pull** (Consumer가 가져감) | **Push** (서버가 밀어줌) |
| **소비 후 데이터** | **유지** (재처리 가능) | **삭제** |
| **저장 방식** | **파일 시스템** | 메모리 (일부 디스크) |
| **순서 보장** | Partition 내에서만 | Queue 내 FIFO |
| **처리량** | **대용량** (초당 수백만) | 중소 규모 |
| **사용 용도** | 이벤트 스트리밍, 로그 수집 | 태스크 큐, 작업 분배 |

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
| **Manager 역할** | 클러스터 메타데이터 관리 |

> ⚠️ **시험 함정**:
> - CDSS 최소 = **4 노드** (1 manager + 3 broker)
> - Broker 수는 **줄일 수 없음** (데이터 안정성 때문)

---

## 🔑 핵심 개념 4: 데이터 분석 플랫폼 서비스

### Data Forest

```
[Data Forest = 데이터 레이크 서비스]

다양한 소스 데이터 → Data Forest → 분석 도구 연동
(Object Storage, DB)     (저장/관리)  (Hadoop, Spark)
```

- 대용량 원시 데이터(Raw Data) 저장 및 관리
- 다양한 분석 도구와 연동

### Data Box Frame

```
[Data Box Frame = 데이터 가상화 서비스]

여러 데이터 소스 → Data Box Frame → 통합 뷰
(DB, 파일, API)    (가상화 레이어)  (단일 인터페이스)
```

- 여러 데이터 소스를 하나의 뷰로 통합
- 실제 데이터 이동 없이 가상화로 접근

### Data Catalog

```
[Data Catalog = 데이터 목록/메타데이터 관리]

"어디에 어떤 데이터가 있나?"
→ 데이터 자산 검색, 분류, 계보(Lineage) 추적
```

- 기업 내 데이터 자산 관리
- 데이터 계보(Lineage) 추적
- 메타데이터 검색

### Data Flow (No-Code ETL) (⭐)

```
[Data Flow = 코딩 없이 ETL 파이프라인 구축]

Extract → Transform → Load

GUI 기반으로 드래그&드롭으로 파이프라인 설계
→ 개발자 없이도 데이터 처리 자동화
```

| 서비스 | 핵심 키워드 |
|--------|-----------|
| Data Forest | 데이터 레이크, 원시 데이터 저장 |
| Data Box Frame | 데이터 가상화, 통합 뷰 |
| Data Catalog | 메타데이터, 데이터 계보 |
| **Data Flow** | **No-Code ETL**, 파이프라인 자동화 |

---

## 📝 시험 대비 체크리스트

**Kafka:**
- [ ] 모델: **Topic** (RabbitMQ는 Queue)
- [ ] 전달 방식: **Pull** (RabbitMQ는 Push)
- [ ] 저장 방식: **파일 시스템**
- [ ] 소비 후: **데이터 유지** (재처리 가능)

**RabbitMQ vs Kafka:**
- [ ] Kafka = Topic + Pull + 파일 시스템 + 삭제 안 됨
- [ ] RabbitMQ = Queue + Push + 소비 후 삭제

**CDSS:**
- [ ] 최소 노드: **4개** (Manager 1 + Broker 3)
- [ ] Broker 수: 증가 ✅ / **감소 ❌**

**데이터 플랫폼:**
- [ ] Data Flow = **No-Code ETL** (코딩 없이 파이프라인)
- [ ] Data Catalog = 메타데이터, 데이터 계보
- [ ] Data Forest = 데이터 레이크

---

## 🧠 암기 핵심 문장

> "**Kafka = Topic + Pull + 파일저장 + 재처리 가능**"  
> "**RabbitMQ = Queue + Push + 소비 후 삭제**"  
> "**CDSS 최소 = 4노드 (Manager 1 + Broker 3), Broker 감소 불가**"

---

*다음 챕터: `23_보안_웹취약점_보안상품.md` → 웹 보안, KMS, Private CA, WAF*
