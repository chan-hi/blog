---
title: "Chapter 21: 데이터 공장 — Cloud Search & Hadoop"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, cloud-search, hadoop, hdfs, yarn, mapreduce]
description: "NCP NCE 자격증 대비 학습 자료 - Cloud Search와 Hadoop 생태계 핵심 구조 정리"
---
> **학습 목표**: Cloud Search 컨테이너 구성/지원 언어, Cloud Hadoop Master 구성/Eco 시스템, HDFS/MapReduce/YARN 동작 원리를 완벽히 이해한다.

---

## 이야기: 빵 데이터가 쌓이다

클라우드빵집 3년 차.
하루에 100만 건의 주문 데이터, 검색 로그, 고객 행동 데이터가 쌓이기 시작했다.

> "이 데이터들을 분석하면 어떤 빵이 어느 시간대에 잘 팔리는지 알 수 있을 텐데..."

재현이 말했다.

> "빅데이터 플랫폼이 필요해. NCP의 Cloud Hadoop 써봤어?
> 수십억 건 데이터도 분산해서 처리할 수 있어."

---

## 🔑 핵심 개념 1: Cloud Search (클라우드 서치)

**Cloud Search** = 검색 서비스를 빠르게 구축할 수 있는 **관리형 검색 엔진 서비스**.

### 기본 구성 (⭐ 시험 포인트!)

```
[Cloud Search 기본 구성]

기본 컨테이너: 2개

┌─────────────┐    ┌─────────────┐
│  Indexer    │    │  Searcher   │
│ (색인 생성) │    │ (검색 처리) │
└─────────────┘    └─────────────┘
      1개                1개
```

| 항목 | 내용 |
|------|------|
| **기본 컨테이너** | **2개** (Indexer 1 + Searcher 1) |
| **지원 언어** | **7개** (한국어, 영어, 일본어, 중국어 등) |

### Cloud Search 특징

| 특징 | 설명 |
|------|------|
| **완성형 검색 서비스** | Elasticsearch 등 직접 구축 없이 사용 |
| **형태소 분석** | 한국어 포함 7개 언어 분석 지원 |
| **실시간 색인** | 데이터 변경 시 즉시 색인 업데이트 |
| **관리형 서비스** | 인프라 관리 불필요 |

---

## 🔑 핵심 개념 2: Cloud Hadoop 기본 구성 (⭐⭐ 빈출!)

**Cloud Hadoop** = 대규모 데이터 분산 처리를 위한 **관리형 Hadoop 클러스터**.

### Hadoop 클러스터 구조

```
[Cloud Hadoop 기본 구성]

┌──────────────────────────────────────────┐
│              Master Node ×2              │
│  (NameNode, ResourceManager, 고가용성)   │
└──────────────────────────────────────────┘
              ↕ 관리
┌──────────┐  ┌──────────┐  ┌──────────┐
│  Data    │  │  Data    │  │  Data    │
│  Node 1  │  │  Node 2  │  │  Node 3  │
│(HDFS +   │  │(HDFS +   │  │(HDFS +   │
│ YARN NM) │  │ YARN NM) │  │ YARN NM) │
└──────────┘  └──────────┘  └──────────┘

Edge Node (선택): SSL VPN 접속으로 클러스터 접근
```

| 항목 | 내용 |
|------|------|
| **Master Node** | **기본 2개** (HA 구성) |
| **주요 데이터 소스** | **Object Storage** (주 데이터 저장소) |
| **Edge Node 접속** | **SSL VPN** 통해 접속 |

> ⚠️ **시험 포인트**: Cloud Hadoop Master Node는 **기본 2개**!  
> 데이터 원본은 **Object Storage** 활용!

---

## 🔑 핵심 개념 3: HDFS (Hadoop Distributed File System)

**HDFS** = Hadoop의 분산 파일 시스템. 대용량 파일을 여러 노드에 나눠 저장.

### HDFS 구성 요소 (⭐⭐ 최빈출!)

```
[HDFS 구조]

NameNode (메타데이터 관리)
"파일 A는 Block1, Block2, Block3으로 나눠져 있고,
 각 블록은 DataNode 1, 2, 3에 있어"
    │
    ├── DataNode 1: [Block1 복사본] [Block3 복사본]
    ├── DataNode 2: [Block1 복사본] [Block2 복사본]
    └── DataNode 3: [Block2 복사본] [Block3 복사본]
    
→ 각 블록 3개씩 복제 (3-copy)
```

| 구성 요소 | 역할 | 시험 포인트 |
|---------|------|-----------|
| **NameNode** | **메타데이터** 관리 (어디에 뭐가 있는지) | - |
| **DataNode** | 실제 데이터 블록 저장 | **3개 복사본** 유지 |
| **Heartbeat** | DataNode → NameNode 상태 보고 | **3초** 단위 |

> ⚠️ **시험 포인트**:
> - NameNode = 메타데이터 (디렉토리 구조)
> - DataNode = 실제 데이터 (3-copy)
> - Heartbeat 주기 = **3초**

---

## 🔑 핵심 개념 4: MapReduce

**MapReduce** = 대용량 데이터를 분산 처리하는 프로그래밍 모델.

```
[MapReduce 처리 과정]

원본 데이터: "크로아상 식빵 크로아상 바게트 식빵 크로아상"

  [Map 단계]
  각 단어를 Key-Value 쌍으로 변환
  크로아상→1, 식빵→1, 크로아상→1, 바게트→1, 식빵→1, 크로아상→1

      ↓

  [Shuffle 단계]
  같은 Key끼리 모으기
  크로아상: [1,1,1]
  식빵: [1,1]
  바게트: [1]

      ↓

  [Reduce 단계]
  각 Key의 값 합산
  크로아상: 3
  식빵: 2
  바게트: 1
```

| 단계 | 역할 |
|------|------|
| **Map** | 입력 데이터를 Key-Value 쌍으로 변환 |
| **Shuffle** | 같은 Key끼리 그룹화 |
| **Reduce** | 그룹화된 데이터를 집계/처리 |

---

## 🔑 핵심 개념 5: YARN (Yet Another Resource Negotiator)

**YARN** = Hadoop의 **자원 관리자**. 클러스터 리소스(CPU, 메모리)를 스케줄링.

```
[YARN 구성]

┌─────────────────────────────┐
│    Resource Manager (1)     │
│   (전체 클러스터 자원 관리) │
└─────────────────────────────┘
         ↕                ↕
┌──────────────┐   ┌──────────────┐
│ Node Manager │   │ Node Manager │
│   (Node 1)   │   │   (Node 2)   │
│ [App Master] │   │              │
└──────────────┘   └──────────────┘
```

| 구성 요소 | 역할 |
|---------|------|
| **Resource Manager** | 전체 클러스터 자원 관리, 스케줄링 |
| **Node Manager** | 각 노드의 자원 사용량 모니터링, 컨테이너 실행 |
| **Application Master** | 개별 애플리케이션 자원 요청 및 관리 |

---

## 🔑 핵심 개념 6: Hadoop Eco System

Hadoop 위에서 돌아가는 다양한 도구들.

| 도구 | 역할 | 한 줄 요약 |
|------|------|-----------|
| **Zookeeper** | 분산 코디네이션 | 클러스터 설정 동기화, Master 선출 |
| **HBase** | NoSQL DB | HDFS 위의 실시간 조회 DB |
| **Flume** | 로그 수집 | 실시간 로그를 HDFS로 전송 |
| **Sqoop** | DB ↔ HDFS | RDBMS ↔ Hadoop 데이터 이전 |
| **Spark** | 인메모리 처리 | MapReduce보다 100배 빠른 처리 |
| **Hive** | SQL-like 쿼리 | HDFS 데이터를 SQL로 조회 |
| **Pig** | 스크립트 처리 | 데이터 흐름 언어로 처리 |
| **Presto** | 대화형 쿼리 | 초고속 SQL 분산 쿼리 엔진 |

```
[클라우드빵집 빅데이터 파이프라인]

주문 DB (MySQL)
    ↓ Sqoop (데이터 이전)
HDFS (대용량 저장)
    ↓ Hive (SQL 분석) / Spark (빠른 처리)
분석 결과
    ↓
시각화 대시보드
```

---

## 📝 시험 대비 체크리스트

**Cloud Search:**
- [ ] 기본 컨테이너: **2개** (Indexer 1 + Searcher 1)
- [ ] 지원 언어: **7개**

**Cloud Hadoop:**
- [ ] Master Node: 기본 **2개** (HA)
- [ ] 주 데이터 소스: **Object Storage**
- [ ] Edge Node 접속: **SSL VPN**

**HDFS:**
- [ ] NameNode: **메타데이터** 관리
- [ ] DataNode: 실제 데이터, **3개 복사본**
- [ ] Heartbeat 주기: **3초**

**MapReduce:**
- [ ] 단계: **Map → Shuffle → Reduce**

**YARN:**
- [ ] Resource Manager: 전체 자원 관리
- [ ] Node Manager: 각 노드 자원 모니터링
- [ ] Application Master: 앱별 자원 요청

**Hadoop Eco:**
- [ ] Flume = 로그 수집 / Sqoop = DB↔HDFS 이전
- [ ] Spark = 인메모리(빠른 처리) / Hive = SQL 조회
- [ ] Zookeeper = 분산 코디네이션

---

## 🧠 암기 핵심 문장

> "**Cloud Search = Indexer + Searcher 2컨테이너, 7개 언어**"  
> "**Cloud Hadoop = Master 2개, Object Storage 연동**"  
> "**HDFS = NameNode(메타) + DataNode(3복사, 3초 Heartbeat)**"  
> "**MapReduce = Map → Shuffle → Reduce**"

---

*다음 챕터: `22_BigData_Kafka_DataStreaming.md` → Kafka, CDSS, 데이터 파이프라인*
