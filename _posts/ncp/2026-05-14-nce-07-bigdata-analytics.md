---
title: "[NCE] 07. 빅데이터 & 분석"
date: 2026-05-14 10:00:00 +0900
categories: [NCP, NCE]
tags: [NCP, NCE, Hadoop, Kafka, CloudSearch, CloudDSS, DataFlow, HDFS, MapReduce, BigData]
description: "Cloud Search, Cloud Hadoop, HDFS, MapReduce, Kafka(CloudDSS), DataFlow, DataCatalog, DataForest 정리"
---

## 핵심 요약

- Cloud Search: Indexer 1대 + Searcher 1대 기본, 7개 언어, 4단계 설정
- Cloud Hadoop: Master 2대 이중화, Worker 최소 2대, SSL VPN으로 Edge Node 접속
- HDFS: NameNode(3초 heartbeat), DataNode(3 copy), MapReduce(2004 구글 발표)
- Kafka: LinkedIn 개발, pull 방식, 파일시스템 저장
- CloudDSS: Manager 1대 + Broker 3대 이상 = 최소 4대, 브로커 감소 불가, 한번에 최대 10대 추가

## Cloud Search

- 웹 콘솔에서 **4단계**만 거치면 검색 환경 구현
- **7개 언어** 지원: 한국어, 영어, 일본어, 중국어(간/번체), 인도네시아어, 태국어
- 네이버의 한국어 형태소 분석 처리기 기반

### 구성

- 프로젝트(도메인)별로 **컨테이너 2대 기본 제공**
  - **Indexer 1대** + **Searcher 1대**
- 상황에 따라 Searcher 컨테이너 추가 프로비저닝 가능 (수분 내)

### 검색 시스템 구성 요소

| 요소 | 역할 |
|------|------|
| 수집 시스템 | 전자화된 문서를 모으는 시스템 |
| 정제 시스템 | 검색에 알맞게 가공 |
| 색인 시스템 | 가공된 문서로 색인 구조 생성 |
| 서빙 시스템 | 사용자 요청에 검색 결과 반환 |

- **Inverted Index**: 색인어별로 해당 단어가 등장하는 문서 목록 저장

## Cloud Hadoop

### HDFS

- **Master = NameNode**, **Slave = DataNode**
- 파일을 블록 단위로 나눠서 복제 저장

| 역할 | 특징 |
|------|------|
| **NameNode** | 메타데이터 관리, **DataNode로부터 3초마다 heartbeat** 수신 |
| **DataNode** | 실제 데이터 저장, 고장 흔하므로 **3 copy** 유지 |

### MapReduce

- **2004년 구글이 발표**한 분산 데이터 처리 프레임워크

| 단계 | 역할 |
|------|------|
| **Map** | 데이터를 Key-Value로 분류/묶음 |
| **Shuffle** | 중간 결과를 Reduce로 전달 |
| **Reduce** | 집계 및 최종 결과 산출 |

### YARN

- 클러스터 **리소스(CPU, 메모리) 관리** + 애플리케이션 실행
- 구성: **Resource Manager** (자원 할당) + **Node Manager** (자원 정보 제공) + **Application Master** (실행 모니터링)

### CloudHadoop 특징

| 항목 | 값 |
|------|-----|
| Master Node | **기본 2대** (이중화) |
| Worker Node | **최소 2대**부터 생성 가능 |
| 노드 스토리지 | **100GB~2000GB** (10GB 단위) 또는 4000GB, 6000GB |
| Edge Node 접속 | **SSL VPN** 통해 접속 |

- **Edge Node**: 외부 접속용 별도 제공, 클러스터 접근 보안성 향상
- **4가지 클러스터 템플릿**: Core / HBase / Presto / Spark Type

### 지원 프레임워크

| 프레임워크 | 분류 | 특징 |
|-----------|------|------|
| **Spark** | 처리 | 인메모리 기반, MapReduce보다 빠름, SQL/ML/스트리밍 지원 |
| **Hive** | 처리 | Facebook 개발, HiveQL(SQL 유사), 데이터 웨어하우스 |
| **Presto** | 처리 | Hive 대비 **CPU 효율성 10배** |
| **Sqoop** | 수집 | RDBMS ↔ HDFS 데이터 전송 |
| **Flume** | 수집 | 로그 스트림 수집 → HDFS 저장 |
| **HBase** | 저장 | HDFS 기반 컬럼 기반 DB |
| **Pig** | 처리 | Pig Latin 언어로 MapReduce 대체 |
| **Zookeeper** | 코디네이터 | 분산 환경 서버 간 조정 |
| **Zeppelin** | 시각화 | Spark 기반 노트북 & 시각화 |
| **HUE** | 관리 | Hadoop 웹 기반 UI |
| **Ambari** | 관리 | 프로비저닝/관리/모니터링, RESTful API |

```
데이터 처리 흐름:
Data Source → 수집(Sqoop/Flume) → 저장(HDFS/HBase) → 분석(Spark/Hive) → 시각화(Zeppelin)
```

## Kafka (CloudDSS)

### Kafka 기본 개념

- **LinkedIn**에서 개발한 분산형 스트리밍 플랫폼
- 대용량 실시간 로그 처리 특화, 기존 메시징 대비 **TPS 우수**
- 메시지를 **파일 시스템에 저장** → 재시작 시 메시지 유실 감소
- **Pub/Sub 모델**: Producer → Kafka Topic → Consumer
- **Pull 방식**: Consumer가 직접 broker에서 메시지 가져감 (push 방식 아님)

### Kafka 구성 요소

| 요소 | 설명 |
|------|------|
| **Producer** | 데이터 생산자 |
| **Consumer** | 메시지 취득 애플리케이션 |
| **Broker** | 메시지 수집/전달 |
| **Topic** | 메시지 구분용 네임 (채널) |
| **Partition** | Topic을 나눈 병렬 처리 단위 |
| **Zookeeper** | 클러스터 메타데이터 관리 |

### Kafka vs RabbitMQ

| 항목 | RabbitMQ | Kafka |
|------|----------|-------|
| 전달 방식 | Queue (1명) | Topic (다수) |
| 메시지 저장 | 배달 후 삭제 | **오래 저장 가능** |
| 처리량 | 빠르지만 대량에 약함 | **초당 수백만 개** 처리 |

### CloudDSS 노드 구성

| 항목 | 값 |
|------|-----|
| 최소 구성 | Manager **1대** + Broker **3대 이상** = **최소 4대** |
| 브로커 감소 | **불가** |
| 한 번에 최대 추가 | **10대** |
| 브로커 추가 시 | **전체 클러스터 재시작** |
| 브로커 타입 변경 | **불가** |

## DataFlow (ETL/Pipeline)

- **완전 관리형 서버리스** 데이터 통합 서비스
- **시각적 인터페이스**로 코딩 없이 ETL 작업 및 데이터 파이프라인 구성
- 연동: DataCatalog, ObjectStorage, Cloud DB for MySQL

## DataCatalog

- 메타데이터 수집 자동화: 스캔 → 스키마 추론 → 테이블 자동 생성
- Cloud DB 및 ObjectStorage 연결
- 태그를 활용한 검색 기능
- 테이블/스키마 변경 이력 관리

## DataForest

- Apache Hadoop 기반 대용량 멀티테넌트 빅데이터 처리 클러스터
- **GPU 리소스 동적 할당** → TensorFlow, PyTorch 딥러닝 학습 가능
- 컨테이너 기반, 온라인 상태에서 동적 확장 가능

## DataboxFrame

- 내/외부 분석자가 안전하게 데이터 분석하는 **데이터 분석 플랫폼**
- **데이터 반출 시 관리자의 타당성 심사 및 승인 필요**

## 시험 암기 포인트

- **Cloud Search**: Indexer 1대 + Searcher 1대 = 2대, 7개 언어, 4단계
- **HDFS**: NameNode 3초 heartbeat, DataNode 3 copy
- **MapReduce**: 2004년 구글 발표
- **Cloud Hadoop**: Master 2대 이중화, Worker 최소 2대, SSL VPN으로 Edge Node 접속, 템플릿 4종
- **Kafka**: LinkedIn 개발, pull 방식, 파일시스템 저장
- **CloudDSS**: Manager 1대 + Broker 3대 이상 = **최소 4대**, 브로커 감소 불가, 추가 최대 10대, 추가 시 전체 재시작

