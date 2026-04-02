---
title: "NCE 자격증 스토리텔링 교재 — 클라우드빵집 CTO의 모험"
date: 2026-04-02 13:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, 학습가이드, 스토리텔링]
description: "NCP NCE 자격증 대비 학습 자료 - 전체 챕터 구성 및 학습법 안내 프롤로그 (AI·DevOps·아키텍처 포함 전 26챕터)"
---

## "클라우드빵집 CTO의 모험"

> NCP(Naver Cloud Platform) NCE 자격증 완벽 대비  
> NCP 공식 문서 기반 | 스토리 학습법

---

## 📖 이 교재를 읽는 법

이 교재는 **민준**이라는 스타트업 창업자가 **클라우드빵집**을 운영하면서 NCP 인프라를 처음부터 구축해 나가는 이야기입니다.

기술 개념이 나올 때마다, 민준이 실제로 겪는 상황과 함께 등장합니다.  
이야기 속에서 자연스럽게 개념을 익히고, 챕터 끝의 **시험 대비 체크리스트**로 완성하세요.

---

## 📚 전체 챕터 설계

### Part 1 — 컴퓨팅 & 스토리지

| 챕터 | 주요 내용 |
|------|-----------|
| [Chapter 1: 하이퍼바이저](../nce-01-hypervisor/) | G1/G2(XEN) vs G3(KVM), CPU/Memory 규칙, OS 스토리지 차이 |
| [Chapter 2: 서버 타입](../nce-02-server-types/) | Micro/Standard/High Memory/High CPU/CPU-Intensive |
| [Chapter 3: 특수 서버](../nce-03-special-servers/) | Bare Metal Server(RAID 1+0/5), GPU(Pass Through) |
| [Chapter 4: 서버 운영](../nce-04-server-operations/) | 서버 상태 관리, 이미지, 스냅샷, NIC, CLI/API |
| [Chapter 5: 보안 ACG·NACL](../nce-05-security-acg-nacl/) | ACG(Stateful/Allow만) vs NACL(Stateless/Deny가능) |
| [Chapter 6: 스토리지 3대장](../nce-06-storage-three/) | Block / File / Object 스토리지 비교 |
| [Chapter 7: Object Storage](../nce-07-object-storage/) | S3 호환, 버킷 1000개, API 10TB 업로드 |
| [Chapter 8: Archive & NAS](../nce-08-archive-nas/) | Archive(swift CLI, 저장 싸고 API 비쌈), NAS(500GB~20TB) |
| [Chapter 9: Backup 전략](../nce-09-backup-strategy/) | 전체/차등/증분 백업, 보관 7일~364일 |

### Part 2 — 컨테이너

| 챕터 | 주요 내용 |
|------|-----------|
| [Chapter 10: Docker](../nce-10-docker/) | VM vs Container, Dockerfile, 레이어 구조 |
| [Chapter 11: Kubernetes](../nce-11-kubernetes/) | K8s 컴포넌트, Pod/ReplicaSet/Deployment, Service |
| [Chapter 12: NKS](../nce-12-nks/) | 관리형 K8s, Master=NCP 관리, Container Registry |

### Part 3 — 네트워크

| 챕터 | 주요 내용 |
|------|-----------|
| [Chapter 13: 네트워크 기초](../nce-13-network-basics/) | OSI 7계층 vs TCP/IP 4계층, CIDR 계산 |
| [Chapter 14: VPC & Transit VPC](../nce-14-vpc-transit-vpc/) | VPC 구조, Peering(1:1) vs Transit VPC(1:N) |
| [Chapter 15: 로드밸런서](../nce-15-load-balancer/) | NLB/ProxyLB/ALB 비교, DSR, 알고리즘 3종 |
| [Chapter 16: 로드밸런서 심화](../nce-16-load-balancer-advanced/) | Sticky Session, Idle Timeout, X-Forwarded 헤더 |
| [Chapter 17: DNS & CDN](../nce-17-dns-cdn/) | DNS 레코드 9종, TTL, Cache Hit/Miss, Global Edge |
| [Chapter 18: 미디어 서비스](../nce-18-media-services/) | Live Station(RTMP→HLS), VOD Station, Image Optimizer |

### Part 4 — 데이터베이스 & 빅데이터

| 챕터 | 주요 내용 |
|------|-----------|
| [Chapter 19: DB MySQL·MSSQL](../nce-19-cloud-db-mysql-mssql/) | MySQL(6TB, 10Replica, 1분 Failover), MSSQL(Log Shipping) |
| [Chapter 20: DB 비교](../nce-20-database-comparison/) | PostgreSQL/Redis/MongoDB 핵심 수치 총정리 |
| [Chapter 21: BigData 기초](../nce-21-cloud-search-hadoop/) | Cloud Search(7개언어), HDFS, MapReduce, YARN, Hadoop |
| [Chapter 22: Kafka & 스트리밍](../nce-22-kafka-data-streaming/) | Kafka vs RabbitMQ, CDSS(최소 4노드), Data Flow |

### Part 5 — 보안

| 챕터 | 주요 내용 |
|------|-----------|
| [Chapter 23: 보안 상품](../nce-23-security-products/) | SQL Injection/XSS/SSRF/XXE, KMS 봉투암호화, Private CA, WAF |

### Part 6 — AI & DevOps & 아키텍처

| 챕터 | 주요 내용 |
|------|-----------|
| [Chapter 24: CLOVA AI 서비스](../nce-24-ai-services/) | Chatbot(6개언어), OCR(Basic/Premium), Speech(STT), Voice(TTS), Papago, Search Trend |
| [Chapter 25: DevOps Source 4총사](../nce-25-devops-source-tools/) | SourceCommit→SourceBuild→SourceDeploy→SourcePipeline CI/CD |
| [Chapter 26: 클라우드 아키텍처](../nce-26-cloud-architecture/) | 3-tier, 모놀리식 vs MSA, Cloud Functions(서버리스), IoT Core |

### 마무리

| | 내용 |
|--|------|
| [최종 체크리스트](../nce-99-final-checklist/) | 전 챕터 핵심 수치 총집합 + 함정 문제 TOP 28 |

---

## 🏠 프롤로그: 클라우드빵집이 문을 열다

2024년 봄, **민준**은 드디어 창업을 결심했다.  
이름하여 **"클라우드빵집"** — 전국 배달 주문을 받는 온라인 베이커리.

문제는 하나였다. **서버**.

> "직접 서버를 사야 하나? 근데 언제 주문이 몰릴지 모르잖아.  
> 서버 터지면 어떡하지? 초기 비용도 너무 비싸고..."

고민하던 민준에게 친구 **지수**가 말했다.

> "야, **NCP** 써봐. 네이버 클라우드 플랫폼.  
> 쓴 만큼만 내고, 콘솔에서 클릭 몇 번이면 서버 뚝딱이야."

민준은 그날부터 NCP 콘솔에 접속했다.  
그리고 깨달았다. 이건 단순한 서버 대여가 아니라, **인프라의 모든 것을 설계하는 일**이라는 것을.

자, 이제 민준과 함께 클라우드빵집의 인프라를 처음부터 만들어보자.

---

## 🗺️ 클라우드빵집 최종 아키텍처

```
                    ┌──────────────────────────────────┐
                    │          인터넷 (Internet)          │
                    └───────────────┬──────────────────┘
                                    │
                    ┌───────────────▼──────────────────┐
                    │      ALB (Application LB, L7)     │
                    └───────────────┬──────────────────┘
                                    │
                    ┌───────────────▼──────────────────┐
                    │        NACL (서브넷 방화벽)         │
                    └───┬───────────────────────────┬──┘
                        │                           │
          ┌─────────────▼──────────┐   ┌────────────▼───────────┐
          │   웹 서버 (G3/KVM)      │   │   GPU 서버 (G3/KVM)    │
          │   Standard 1:4         │   │   딥러닝 / AI 모델      │
          │   Auto Scaling 적용     │   │   Pass Through 방식    │
          │   ACG 적용              │   │   ACG 적용             │
          └─────────────┬──────────┘   └────────────────────────┘
                        │
          ┌─────────────▼──────────┐
          │   DB 서버 (G3/KVM)      │
          │   High Memory 1:8      │
          │   Cloud DB MySQL       │
          │   (6TB, Replica 10대)  │
          └─────────────┬──────────┘
                        │
          ┌─────────────▼──────────────────────────────────────┐
          │                    스토리지 계층                      │
          │  Object Storage (이미지/영상)   ← AWS CLI            │
          │  Archive Storage (장기 보관)    ← OpenStack swift    │
          │  NAS (공유 파일)  NFS/CIFS      ← 500GB ~ 20TB      │
          │  Backup (증분+전체)             ← 7일 ~ 364일        │
          └────────────────────────────────────────────────────┘

                    ┌──────────────────────────────────┐
                    │         AI 서비스 레이어            │
                    │  CLOVA Chatbot  — 고객 문의 자동화  │
                    │  CLOVA OCR      — 영수증 자동 인식  │
                    │  CLOVA Speech   — 음성 주문 처리    │
                    │  CLOVA Voice    — 안내 방송 TTS     │
                    │  Papago         — 다국어 메뉴 번역  │
                    └──────────────────────────────────┘

                    ┌──────────────────────────────────┐
                    │        DevOps 파이프라인           │
                    │  SourceCommit  → 코드 저장 (Git)   │
                    │       ↓                           │
                    │  SourceBuild   → 자동 빌드         │
                    │       ↓                           │
                    │  SourceDeploy  → 자동 배포         │
                    │  (SourcePipeline으로 전체 자동화)  │
                    └──────────────────────────────────┘
```

---

## 📈 민준의 성장 로드맵

```
Day 1  서버 첫 생성 (하이퍼바이저, 서버 타입 선택)
  ↓
Day 7  보안 설계 (ACG + NACL + VPC)
  ↓
Day 14 스토리지 구성 (Object Storage + NAS + Backup)
  ↓
Day 21 컨테이너 전환 (Docker → K8s → NKS)
  ↓
Day 30 네트워크 완성 (ALB + DNS + CDN)
  ↓
Day 45 DB 고가용성 (MySQL Failover + Redis Cache)
  ↓
Day 60 빅데이터 구축 (Hadoop + Kafka + Cloud Search)
  ↓
Day 75 보안 강화 (KMS + WAF + Private CA)
  ↓
Day 90 AI 도입 (CLOVA Chatbot + OCR + Speech)
  ↓
Day 100 DevOps 완성 (SourcePipeline CI/CD 자동화)
  ↓
Day 110 아키텍처 고도화 (MSA + Cloud Functions + IoT)
  ↓
시험 합격! 🎉
```

---

*다음 챕터: Chapter 1 — 서버를 처음 만드는 순간, 하이퍼바이저를 선택하라*
