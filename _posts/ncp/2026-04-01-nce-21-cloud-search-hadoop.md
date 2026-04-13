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

> "그전에 검색 기능부터 붙여보자. Cloud Search 쓰면 Elasticsearch 직접 구축 안 해도 돼.
> 4단계만 거치면 바로 검색 환경이 만들어진다고."

---

## 🔑 핵심 개념 1: Cloud Search (클라우드 서치)

**Cloud Search** = 검색 서비스를 빠르게 구축할 수 있는 **관리형 검색 엔진 서비스**.

Elasticsearch, Solr, Naver(Cloud) 모두 **Lucene**이라는 검색 엔진을 기반으로 한다. 검색 시스템은 수집 → 형태소 분석 → DSL(Domain Specific Language) → 결과 산출이라는 복잡한 구조로 이루어져 있는데, Cloud Search는 이 모든 것을 관리형으로 제공한다.

### Cloud Search 4단계 구성 (⭐ 시험 포인트!)

```
[Cloud Search 웹 콘솔 4단계]

1단계: 검색 환경 설정
   → 인덱싱 옵션, 자동완성 검색어, 스테밍 수준 제어,
     검색 결과 정렬 제어 등 설정

2단계: 검색 대상 문서 업로드
   → Object Storage에서 업로드
   → 데이터베이스에서 업로드

3단계: 문서 인덱싱
   → 색인어 추출 + Inverted Index 테이블 생성
   → 형태소 분석기로 불용어 처리 및 색인 구조 생성

4단계: 검색 질의 (API)
   → REST API로 쿼리 전송 → 결과 제공
```

### 기본 컨테이너 구성

```
[Cloud Search 기본 구성]

기본 컨테이너: 2개

┌─────────────┐    ┌─────────────┐
│  Indexer    │    │  Searcher   │
│ (색인 생성) │    │ (검색 처리) │
└─────────────┘    └─────────────┘
      1개                1개

→ 추후 상황에 따라 Searcher 컨테이너 추가 프로비저닝 가능
→ 모니터링 상황에 따라 수분 내 확장/축소 가능
```

| 항목 | 내용 |
|------|------|
| **기본 컨테이너** | **2개** (Indexer 1 + Searcher 1) |
| **지원 언어** | **7개** (한국어, 영어, 일본어, 중국어 간체, 중국어 번체, 인도네시아어, 태국어) |
| **프로비저닝 단위** | 프로젝트 단위 (도메인) |

### Cloud Search 주요 기능

| 기능 | 설명 |
|------|------|
| **도메인 관리** | 용도에 따라 도메인 생성·삭제 |
| **섹션 관리** | 검색 대상 문서 형식 설정 |
| **색인 관리** | 검색 쿼리 목적에 따라 조회할 섹션 정의 |
| **검색 관리** | 다국어 사전, 동의어 사전 설정으로 정확도 향상 |
| **랭킹 관리** | 문서에 가중치를 설정하여 검색 결과 순서 제어 |
| **자동완성** | 자동완성 검색어 후보를 섹션·색인으로 설정 |
| **불용어 관리** | 검색 쿼리에서 제외될 단어 설정 |
| **모니터링** | 과금 관련 주요 지표 조회 |
| **쿼리 분석** | 인기 검색어, 시간별·지역별 쿼리 통계 조회 |

### 인덱싱(Indexing)이란?

색인이란 **대량 문서에서 빠르게 원하는 것을 찾을 수 있는 자료구조**다.

```
[색인 생성 과정]

Document (수집·정제된 문서)
    ↓ 형태소 분석기
색인어 추출 + 불용어 처리
    ↓
Inverted Index 테이블 생성
    ↓
검색 엔진에 색인 자료구조 저장

[검색 질의 과정]
사용자 질의 → 형태소 분석기 → 로직 서버 → 검색 엔진 → 결과 반환
```

### Cloud Search 특징

| 특징 | 설명 |
|------|------|
| **완성형 검색 서비스** | Elasticsearch 등 직접 구축 없이 사용 |
| **형태소 분석** | 한국어 포함 7개 언어 분석 지원 |
| **실시간 색인** | 데이터 변경 시 즉시 색인 업데이트 |
| **관리형 서비스** | 인프라 관리 불필요 |
| **Semantic 검색** | 자동완성, 랭킹, 다국어 지원 |
| **REST API** | 관련 기능 모두 REST API로 제공 |

---

## 🔑 핵심 개념 2: 빅데이터 처리 방식

Cloud Hadoop을 제대로 이해하려면 빅데이터 처리 방식부터 알아야 한다.

| 종류 | 내용 | 대표 도구 |
|------|------|-----------|
| **배치 처리** | 초·분·시간·일 단위로 일괄 처리 | MapReduce, Hive, Pig |
| **대화형 처리** | 원하는 질의에 대해 수초 내 답을 얻는 방식 | Impala, Pig, Spark |
| **실시간 처리** | 즉각적으로 데이터 저장·분석, 데이터 스트림 방식 | Spark Streaming, Storm |

> 예: 배치 처리 = 급여·재고 처리, 주간/월간 청구 / 실시간 처리 = 차량 공유 서비스 고객 매칭, 금융 거래 이상 여부 탐지

---

## 🔑 핵심 개념 3: Cloud Hadoop 기본 구성 (⭐⭐ 빈출!)

**Cloud Hadoop** = 대규모 데이터 분산 처리를 위한 **관리형 Hadoop 클러스터**.
"빅데이터를 쉽고 빠르게 처리할 수 있는 오픈소스 기반의 분석 서비스"

### Hadoop 클러스터 구조

```
[Cloud Hadoop 기본 구성]

       SSL VPN Client
            ↓
       [Edge Node]  ← 외부에서 클러스터 접근 시 반드시 거침
       Public or Private Subnet
            ↓
┌──────────────────────────────────────────┐
│           Master Node ×2 (기본)          │
│  NameNode + ResourceManager + 이중화     │
└──────────────────────────────────────────┘
              ↕ 관리 (Private Subnet)
┌──────────┐  ┌──────────┐  ┌──────────┐
│  Worker  │  │  Worker  │  │  Worker  │
│  Node 1  │  │  Node 2  │  │  Node 3  │
│(HDFS +   │  │(HDFS +   │  │(HDFS +   │
│ YARN NM) │  │ YARN NM) │  │ YARN NM) │
└──────────┘  └──────────┘  └──────────┘
           (최소 2대부터 생성 가능)

DataSource: Object Storage (Primary) 또는 HDFS
```

| 항목 | 내용 |
|------|------|
| **Master Node** | **기본 2개** (이중화, HA 구성) |
| **Worker Node** | **최소 2대**부터 생성 가능 |
| **각 노드 스토리지** | 100GB~2000GB (10GB 단위) 또는 4000GB, 6000GB |
| **주요 데이터 소스** | **Object Storage** (Primary Datasource) |
| **Edge Node 접속** | **SSL VPN** 통해 접속 |
| **클러스터 접근 권한** | 루트(root) 접근 권한 제공 |

> ⚠️ **시험 포인트**: Cloud Hadoop Master Node는 **기본 2개**!
> 데이터 원본은 **Object Storage** 활용! Worker Node는 **최소 2대**!

### Cloud Hadoop 4가지 클러스터 템플릿

| 타입 | 구성 |
|------|------|
| **Core** | 기본 Hadoop 구성 |
| **HBase** | HBase 포함 구성 |
| **Presto** | Presto 포함 구성 |
| **Spark** | Spark 포함 구성 |

---

## 🔑 핵심 개념 4: Hadoop 아키텍처 (노드 역할)

```
[Hadoop 아키텍처 구성]

MasterServer / NameNode
  ├── MapReduce: 분산 처리
  ├── YARN: 자원 스케줄링 관리
  └── HDFS: 분산 파일 시스템

DataNode (Worker Node)
  ├── MapReduce: 분산 처리
  ├── YARN: 자원 스케줄링 관리
  └── HDFS: 분산 파일 시스템

DataNode (Secondary Node)
  ├── MapReduce: 분산 처리
  ├── YARN: 자원 스케줄링 관리
  └── HDFS: 분산 파일 시스템
```

각 노드에서 MapReduce, YARN, HDFS가 함께 동작하며, Master가 전체를 조율하고 DataNode(Worker/Secondary)가 실제 데이터를 저장·처리한다.

---

## 🔑 핵심 개념 5: HDFS (Hadoop Distributed File System)

**HDFS** = Hadoop의 분산 파일 시스템. 대용량 파일을 여러 노드에 나눠 저장.
구글 파일 시스템(GFS)을 대체하기 위해 설계된 분산 파일 시스템.

### HDFS 구성 요소 (⭐⭐ 최빈출!)

```
[HDFS 구조]

          ┌──────────────┐   ┌──────────────────┐
          │   NameNode   │   │  Secondary Node  │
          │  (Master)    │   │  (보조 NameNode) │
          │  - 메타데이터│   └──────────────────┘
          │  - 파일 위치 │
          └──────────────┘
                 │ 3초마다 Heartbeat 수신
    ┌────────────┼────────────┐
    ↓            ↓            ↓
DataNode 1   DataNode 2   DataNode 3
[Block1 복사] [Block1 복사] [Block2 복사]
[Block3 복사] [Block2 복사] [Block3 복사]

→ 각 블록 3개씩 복제 (3-copy)
```

| 구성 요소 | 역할 | 시험 포인트 |
|---------|------|-----------|
| **NameNode** | **메타데이터** 관리 (파일 시스템 유지), DataNode 모니터링 | - |
| **Secondary Node** | NameNode 보조, 메타데이터 백업 | - |
| **DataNode** | 실제 데이터 블록 저장 | **3개 복사본** 유지 |
| **Heartbeat** | DataNode → NameNode 상태 보고 | **3초** 단위 |

#### NameNode 상세 역할
- **메타데이터 관리**: 파일 시스템을 유지하기 위한 메타데이터 관리
- **DataNode 모니터링**: 3초마다 Heartbeat 전송 → 생존 신고
- **블록 관리**: 장애가 발생한 DataNode의 블록 교체

#### DataNode 상세 역할
- 실제 저장되는 데이터 보관
- DataNode는 고장이 흔하게 발생하므로 **3-copy 유지**
- 분산 파일 시스템의 파일을 블록 단위로 나눠서 복제 저장
- 경우에 따라 Object Storage를 저장 장치로 사용 가능

> ⚠️ **시험 포인트**:
> - NameNode = 메타데이터 (디렉토리 구조)
> - DataNode = 실제 데이터 (3-copy)
> - Heartbeat 주기 = **3초**
> - Secondary Node = NameNode 보조 역할

---

## 🔑 핵심 개념 6: MapReduce

**MapReduce** = 대용량 데이터를 분산 처리하는 프로그래밍 모델.
구글이 수집한 문서와 로그 등 방대한 데이터를 분석하기 위해 **2004년**에 발표.

```
[MapReduce 처리 과정]

원본 데이터: "크로아상 식빵 크로아상 바게트 식빵 크로아상"

  [Map 단계]
  정렬되지 않은 데이터를 Key-Value 연관성 있는 데이터로 분류
  크로아상→1, 식빵→1, 크로아상→1, 바게트→1, 식빵→1, 크로아상→1

      ↓

  [Shuffle/Sort 단계]
  같은 Key끼리 모으기 (중간 결과를 Reduce로 전달)
  크로아상: [1,1,1]
  식빵: [1,1]
  바게트: [1]

      ↓

  [Reduce 단계]
  리스트에서 원하는 데이터를 찾아서 집계
  크로아상: 3
  식빵: 2
  바게트: 1
```

| 단계 | 역할 |
|------|------|
| **Map** | 입력 데이터를 Key-Value 쌍으로 변환, 속성이 같은 형태로 분류 |
| **Shuffle/Sort** | 같은 Key끼리 그룹화, 중간 결과를 Reduce로 전달 |
| **Reduce** | 그룹화된 데이터에서 원하는 값을 찾아 집계/처리 |

MapReduce는 방대한 양의 데이터를 **노드에 병렬화하여 처리**하기 위한 프레임워크다.

---

## 🔑 핵심 개념 7: YARN (Yet Another Resource Negotiator)

**YARN** = Hadoop의 **자원 관리자**. 클러스터 리소스(CPU, 메모리 등)를 관리하고 애플리케이션을 실행.

- 여러 개의 프레임워크(MapReduce, Spark 등)를 수행하는 동안 **동적으로 자원을 스케줄링**
- 클러스터 자원을 **컨테이너**로 분할
- 실행 중인 컨테이너 모니터링
- HDFS 상단에서 빅데이터 애플리케이션들을 실행하는 **대용량 분산 운영체제 역할**

```
[YARN 구성]

┌─────────────────────────────────────┐
│        Resource Manager (RM)        │
│     (전체 클러스터 자원 관리·할당)  │
└─────────────────────────────────────┘
         ↕                ↕                ↕
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Node Manager │  │ Node Manager │  │ Node Manager │
│   (Node 1)   │  │   (Node 2)   │  │   (Node 3)   │
│[App Master]  │  │[container]   │  │[container]   │
│[container]   │  │[container]   │  │              │
└──────────────┘  └──────────────┘  └──────────────┘
```

| 구성 요소 | 역할 |
|---------|------|
| **Resource Manager (RM)** | 자원 할당, 전체 클러스터 자원 관리 및 스케줄링 |
| **Node Manager (NM)** | Resource Manager에게 자원 정보 제공, 각 노드 자원 모니터링 |
| **Application Master (AM)** | 프로세스 실행 모니터링, 자원 요청 |

---

## 🔑 핵심 개념 8: Hadoop Eco System

Hadoop 위에서 돌아가는 다양한 도구들. BigData를 **수집 → 저장 → 처리 → 분석/시각화**하는 흐름으로 처리.

| 구성 요소 | 서브 프로젝트 | 설명 |
|---------|-------------|------|
| **코디네이터** | **Zookeeper** | 분산 환경에서 서버 간 상호 조정이 필요한 다양한 서비스 제공, Master 선출 |
| **리소스 관리** | **YARN** | 데이터 처리 작업 실행을 위한 클러스터 지원, 스케줄링 프레임워크 |
| **데이터 저장** | **HBase** | HDFS 기반, 컬럼 기반 데이터베이스 (NoSQL) |
| **데이터 수집** | **Flume** | 에이전트로부터 데이터를 받는 콜렉터 |
| **데이터 수집** | **Sqoop** | RDBMS와 HDFS 간 대용량 데이터 전송 솔루션 |
| **데이터 처리** | **Spark** | 인메모리 기반 범용 데이터 처리 플랫폼, 대규모 데이터 처리 |
| **데이터 처리** | **Pig** | 복잡한 MapReduce 프로그래밍을 대체할 '피그 라틴' 언어 제공 |
| **데이터 처리** | **Hive** | SQL로 분산 스토리지의 대규모 데이터 읽기·쓰기·관리, 데이터 웨어하우스 |
| **데이터 시각화** | **Zeppelin** | 빅데이터 분석 웹 기반 분석 도구 |
| **관리 도구** | **Ambari** | Hadoop 클러스터를 프로비저닝·관리·모니터링·보호하는 관리 플랫폼 |
| **관리 도구** | **HUE** | Hadoop User Experience, 웹 기반 SQL Cloud Editor |

### 수집 도구 상세

#### Sqoop (SQL to Hadoop)

Hadoop과 **관계형 데이터베이스 간 데이터 전송**을 위해 설계된 오픈소스 소프트웨어.

```
[Sqoop 동작]

RDBMS (MySQL, Oracle, PostgreSQL 등)
    ↕  import / export
Hadoop File System (HDFS)
    → Hive, Pig, HBase 등으로 바로 이전도 가능
```

- MapReduce를 기반으로 구현된 데이터 적재 프로그램
- 간단한 CLI(Command Line Interface)로 특정 테이블 또는 조건에 맞는 데이터를 HDFS로 이전
- 반대로 HDFS에 저장된 데이터를 RDBMS로 내보내기(Export)도 가능
- 모든 적재 과정을 **자동화**하고 **병렬 처리** 방식, **고장 방지 능력(fault tolerance)** 지원
- **Sqoop 1**: 클라이언트 방식 / **Sqoop 2**: 기존 방식 + Server side 방식 추가 (2009년 첫 버전 출시)

#### Flume

연속적으로 생성되는 데이터 스트림을 수집하고 전송해서 HDFS에 저장하는 도구.
로그 파일, SNS 미디어, 이메일, 메시지 등 데이터 스트림을 HDFS에 저장하는 **수집 전용 솔루션**.

```
[Flume 구성]

외부 데이터 소스
    ↓
Source (에이전트) → Channel (버퍼) → Sink (에이전트)
                    메모리/디스크      HDFS, 로컬 파일 등
```

- **Source**: 외부 데이터 소스에 설치되는 에이전트, 데이터 수신 후 채널로 전달, 하나 이상의 채널로 전달 가능
- **Channel**: 소스와 싱크 간 데이터를 받는 통로, 소스와 싱크의 속도를 조절하는 버퍼, 메모리에 저장하며 네트워크 장애 유실 방지를 위해 디스크 저장 설정도 가능
- **Sink**: 데이터 목적지에 설치되는 에이전트, HDFS·로컬 파일 등에 전달, 하나의 싱크는 한 채널에서만 데이터를 전달받을 수 있음

### 처리 도구 상세

#### Hive (Apache Hive)
- 페이스북 주도로 개발 (현재 Netflix 등에서도 사용)
- Hadoop에 저장된 데이터를 **SQL과 유사한 HiveQL**을 사용하여 처리
- DBA가 빅데이터를 분석할 때 유용한 도구
- RDB의 DB·테이블과 같은 형태로 HDFS에 저장된 데이터의 구조 정의 가능

#### Pig
- 데이터 분석을 프로그래밍할 수 있는 대용량 데이터셋 분석 플랫폼
- MapReduce에서 처리할 수 없는 **조인(Join)** 같은 연산을 지원

#### Spark (Apache Spark)
- 데이터 처리부터 시각화까지 가능한 프레임워크
- **메모리 기반(인메모리)** 데이터 처리로 MapReduce보다 빠른 속도
- Java, Python, Scala를 통해 작성 가능
- MapReduce뿐만 아니라 **SQL 쿼리, 스트리밍 데이터, ML, 그래프 알고리즘** 지원
- UC 버클리 AMPLab에서 개발 후 Apache 소프트웨어 재단에 기부

#### Presto (Apache Presto)
- 페이스북이 개발한 빅데이터 분석 도구, **분산 SQL 쿼리 엔진**
- 기존 Hive/MapReduce 대비 CPU 효율성과 대기시간이 **10배 빠름**
- 2013년 11월 아파치 라이선스로 공개

#### HBase (Apache HBase)
- Hadoop 플랫폼을 위한 공개 비관계형 분산 데이터베이스
- 구글의 BigTable을 본보기로 삼아 Java로 작성
- HDFS 위에서 동작

#### Zeppelin Notebook
- Apache Spark을 기반으로 한 **웹 기반 노트북 & 시각화 툴**

#### Ambari (Apache Ambari)
- Hadoop 클러스터를 **프로비저닝, 관리, 모니터링, 보호**할 수 있는 관리 플랫폼
- RESTful API를 통해 Web UI 제공, 직관적이고 customizing 용이
- Hadoop을 기존 엔터프라이즈 인프라와 통합 지원

### 빅데이터 처리 흐름도

```
[BigData Eco Flow]

DataSource    → DataCollecting → DataStore → DataAnalysis → Visualization
(Sensor, IoT)   (Flume, Sqoop)   (HDFS,      (Hive, Spark,  (R, Python,
                                  HBase,       Presto)        Zeppelin)
                                  Object
                                  Storage)
                                     ↑
                                  Hadoop
                                  (MapReduce)
```

```
[클라우드빵집 빅데이터 파이프라인]

주문 DB (MySQL)
    ↓ Sqoop (RDBMS → HDFS 데이터 이전)
HDFS (대용량 저장)
    ↓ Hive (SQL 분석) / Spark (빠른 처리)
분석 결과
    ↓
시각화 대시보드 (Zeppelin)
```

---

## 🔑 핵심 개념 9: Lab 14 — Cloud Hadoop 구성 실습

교재에 소개된 실습 구성 순서:

```
[Lab 14 실습 단계]

1단계: Object Storage 구성
   → Cloud Hadoop의 Primary Datasource로 사용할 버킷 생성

2단계: Cloud Hadoop 구성
   → 클러스터 타입 선택 (Core/HBase/Presto/Spark)
   → Master Node 2대 + Worker Node 최소 2대 설정
   → Object Storage 연동 설정

3단계: Hadoop test data load 실습
   → 테스트 데이터를 HDFS 또는 Object Storage에 업로드

4단계: Query를 통한 data table 생성 확인
   → Zeppelin 웹 노트북에서 쿼리 실행
   → "Welcome to Zeppelin!" 화면에서 분석 작업 수행
```

---

## 📝 시험 대비 체크리스트

**Cloud Search:**
- [ ] 기본 컨테이너: **2개** (Indexer 1 + Searcher 1)
- [ ] 지원 언어: **7개** (한국어, 영어, 일본어, 중국어 간·번체, 인도네시아어, 태국어)
- [ ] 구성 단계: **4단계** (환경 설정 → 문서 업로드 → 인덱싱 → 검색 질의)
- [ ] Searcher 컨테이너: 추후 추가 프로비저닝 가능

**Cloud Hadoop:**
- [ ] Master Node: 기본 **2개** (이중화/HA)
- [ ] Worker Node: 최소 **2대**부터 생성 가능
- [ ] 노드 스토리지: 100GB~2000GB (10GB 단위), 또는 4000GB·6000GB
- [ ] 주 데이터 소스: **Object Storage** (Primary Datasource)
- [ ] Edge Node 접속: **SSL VPN**
- [ ] 클러스터 템플릿: **4가지** (Core, HBase, Presto, Spark)

**HDFS:**
- [ ] NameNode: **메타데이터** 관리 + DataNode 모니터링
- [ ] Secondary Node: NameNode 보조 역할
- [ ] DataNode: 실제 데이터, **3개 복사본**
- [ ] Heartbeat 주기: **3초**

**MapReduce:**
- [ ] 구글이 **2004년** 발표
- [ ] 단계: **Map → Shuffle/Sort → Reduce**
- [ ] 노드에 **병렬화**하여 처리하는 프레임워크

**YARN:**
- [ ] Resource Manager (RM): 자원 할당·관리
- [ ] Node Manager (NM): RM에 자원 정보 제공
- [ ] Application Master (AM): 프로세스 실행 모니터링, 자원 요청
- [ ] 클러스터 자원을 **컨테이너**로 분할

**Hadoop Eco:**
- [ ] **Flume** = 연속 데이터 스트림 수집 → HDFS 저장
- [ ] **Sqoop** = RDBMS ↔ HDFS 대용량 데이터 전송 (MapReduce 기반, CLI 지원)
- [ ] **Spark** = 인메모리(빠른 처리), MapReduce보다 빠름
- [ ] **Hive** = HiveQL(SQL-like), DBA용 빅데이터 분석, 페이스북 개발
- [ ] **Pig** = 피그 라틴 언어, Join 등 복잡한 연산 지원
- [ ] **Presto** = 분산 SQL, Hive 대비 **10배 빠름**, 페이스북 개발
- [ ] **Zookeeper** = 분산 코디네이션, Master 선출
- [ ] **HBase** = HDFS 기반 컬럼형 NoSQL DB, 구글 BigTable 참고
- [ ] **Zeppelin** = 웹 기반 노트북·시각화 (Spark 기반)
- [ ] **Ambari** = 클러스터 프로비저닝·관리·모니터링 플랫폼

---

## 🧠 암기 핵심 문장

> "**Cloud Search = 4단계(설정→업로드→인덱싱→질의), Indexer+Searcher 2컨테이너, 7개 언어**"
> "**Cloud Hadoop = Master 2개(HA), Worker 최소 2대, Object Storage 연동, SSL VPN**"
> "**HDFS = NameNode(메타) + Secondary Node(보조) + DataNode(3복사, 3초 Heartbeat)**"
> "**MapReduce = 2004년 구글 발표, Map → Shuffle/Sort → Reduce**"
> "**YARN = RM(자원할당) + NM(자원정보) + AM(자원요청), 컨테이너로 분할**"
> "**Sqoop = RDBMS↔HDFS, Flume = 스트림→HDFS, Presto = Hive보다 10배 빠름**"

---

*다음 챕터: `22_BigData_Kafka_DataStreaming.md` → Kafka, CDSS, 데이터 파이프라인*
