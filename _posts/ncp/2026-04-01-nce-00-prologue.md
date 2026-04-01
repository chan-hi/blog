---
title: "NCE 자격증 스토리텔링 교재 — 클라우드빵집 CTO의 모험"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, 학습가이드, 스토리텔링]
description: "NCP NCE 자격증 대비 학습 자료 - 전체 챕터 구성 및 학습법 안내 프롤로그"
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

| 파일명 | 챕터 | 주요 내용 |
|--------|------|-----------|
| `00_프롤로그` | 프롤로그 | 교재 소개, 학습법, 전체 구조 |
| `01_하이퍼바이저` | Chapter 1 | G1/G2(XEN) vs G3(KVM), CPU/Memory 규칙, OS 스토리지 차이 |
| `02_서버타입` | Chapter 2 | Micro/Standard/High Memory/High CPU/CPU-Intensive |
| `03_특수서버` | Chapter 3 | Bare Metal Server, GPU Server |
| `04_서버운영` | Chapter 4 | 서버 상태 관리, 이미지, 스냅샷, NIC, CLI/API |
| `05_보안_ACG_NACL` | Chapter 5 | ACG vs NACL, Stateful/Stateless |
| `06_스토리지3대장` | Chapter 6 | Block / File / Object 스토리지 비교 |
| `07_ObjectStorage` | Chapter 7 | S3 호환, 제한 사항, 권한 관리 |
| `08_Archive_NAS` | Chapter 8 | Archive Storage 비용 구조, NAS 마운트 |
| `09_Backup전략` | Chapter 9 | 전체/차등/증분 백업 완전 정복 |
| `10_Docker` | Chapter 10 | VM vs Container, Dockerfile, 라이프사이클 |
| `11_Kubernetes` | Chapter 11 | K8s 컴포넌트, Pod/ReplicaSet/Deployment, Service |
| `12_NKS` | Chapter 12 | NKS 관리형 K8s, NCP 서비스 통합 |
| `13_네트워크기초` | Chapter 13 | OSI 7계층 vs TCP/IP 4계층, CIDR 계산 |
| `14_VPC_TransitVPC` | Chapter 14 | VPC 구조, Peering(1:1) vs Transit VPC(1:N) |
| `15_로드밸런서` | Chapter 15 | NLB/ProxyLB/ALB 비교, DSR, 알고리즘 3종 |
| `16_로드밸런서_심화` | Chapter 16 | Sticky Session, Idle Timeout, X-Forwarded 헤더 |
| `17_DNS_CDN` | Chapter 17 | DNS 레코드 9종, TTL, Cache Hit/Miss, Global Edge |
| `18_미디어서비스` | Chapter 18 | Live Station, VOD Station, Image Optimizer |
| `19_데이터베이스_MySQL_MSSQL` | Chapter 19 | MySQL(6TB, 10Replica), MSSQL(Log Shipping, BI/Batch) |
| `20_데이터베이스_비교` | Chapter 20 | PostgreSQL/Redis/MongoDB + DB 종합 비교표 |
| `21_BigData_CloudSearch_Hadoop` | Chapter 21 | Cloud Search, HDFS, MapReduce, YARN, Hadoop Eco |
| `22_BigData_Kafka_DataStreaming` | Chapter 22 | Kafka vs RabbitMQ, CDSS(최소 4노드), Data Flow |
| `23_보안_웹취약점_보안상품` | Chapter 23 | SQL Injection/XSS/SSRF/XXE, KMS, Private CA, WAF |
| `99_총정리` | 최종 정리 | 핵심 수치 총집합, 시험 직전 체크리스트 |

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

## 🗺️ 클라우드빵집 최종 아키텍처 (스포일러)

```
                    ┌─────────────────────────────────┐
                    │         인터넷 (Internet)         │
                    └──────────────┬──────────────────┘
                                   │
                    ┌──────────────▼──────────────────┐
                    │    NACL (서브넷 방화벽)            │
                    └──────────────┬──────────────────┘
                                   │
         ┌─────────────────────────┼─────────────────────────┐
         │                         │                         │
┌────────▼────────┐     ┌──────────▼──────────┐   ┌─────────▼────────┐
│  웹 서버 (G3)    │     │   DB 서버 (G3)       │   │  GPU 서버 (G3)   │
│  Standard 1:4   │     │  High Memory 1:8     │   │  딥러닝 모델     │
│  ACG 적용       │     │  ACG 적용            │   │  Pass Through    │
└─────────────────┘     └─────────────────────┘   └──────────────────┘
         │                         │
         │              ┌──────────▼──────────┐
         │              │   NAS (공유 스토리지)  │
         │              │  NFS 마운트 / 500GB+ │
         │              └─────────────────────┘
         │
┌────────▼────────────────────────────────────────────────────┐
│                    스토리지 계층                              │
│  Object Storage (이미지/영상) ← AWS CLI                      │
│  Archive Storage (장기 보관)  ← OpenStack swift CLI          │
│  Backup (증분+전체)           ← 7일~364일 보관               │
└─────────────────────────────────────────────────────────────┘
```

---

*다음 챕터: `01_하이퍼바이저.md` → 서버를 처음 만드는 순간*
