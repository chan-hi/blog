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

## 🎯 NCE 자격증 — 시험 코드와 과목 구성

민준이 목표로 하는 **NCE(NAVER Cloud Platform Certified Expert)**는 네이버 클라우드 플랫폼 인증 체계의 최상위 과정입니다.

### 네이버 클라우드 플랫폼 인증 체계 전체 구조

| 인증 등급 | 시험 코드 | 과목명 | 수준 |
|----------|----------|--------|------|
| **NCA** (Associate) | 100 | Overview / Compute / Storage / Database / Network / Media | 기본적인 이해, 인프라 구성 가능 |
| **NCP** (Professional) | 200 | Overview / Compute / Storage | 전문 지식, 독자적 인프라 구축 + 트러블슈팅 |
| **NCP** (Professional) | 202 | Network / Media / Database / Management / Analytics | 동일 |
| **NCP** (Professional) | 207 | Troubleshooting | 동일 |
| **NCP-AI** (Certified AI) | 210 | AI | 네이버 AI 서비스 이해 + HyperCLOVA X 기반 CLOVA Studio |
| **NCE** (Expert) | 301 | Compute / Storage | 전문 지식, 트러블슈팅·커스터마이징 가능 |
| **NCE** (Expert) | 302 | Network / Media | 동일 |
| **NCE** (Expert) | 303 | Database / Management / Analytics / Security / Application / AI | 동일 |
| **NCE** (Expert) | 305 | Architect / Advanced (Hybrid / Global) | 동일 |
| **NCE-AI** (Expert AI) | 310 | AI | RAG 구성, HyperCLOVA X 엔진 기반 챗봇, 비즈니스 적용 가능 |

> 기술자격시험은 **매주 월요일 오전 10시** 시험 오픈, **화·수·목** 시험 시행. 모든 시험은 **온라인 응시**로 진행됩니다.

### NCE 시험은 4개 시험(301·302·303·305)으로 구성

NCE는 단순히 서버 띄우는 법을 묻는 시험이 아닙니다. **트러블슈팅**과 **커스터마이징**, 나아가 **하이브리드·글로벌 아키텍처 설계**까지 요구합니다. 민준이 이 교재로 공부하는 이유가 바로 여기에 있습니다.

### NCE 교육 목차 (공식 커리큘럼)

- Overview (클라우드 컴퓨팅 개념, NCP 전반 소개)
- Compute 상품에 대한 이해
- Container 환경에 대한 이해
- Network 상품에 대한 이해
- 미디어 상품에 대한 이해 (Image Optimizer, Live Station, Media Connect Center 등)
- Database 상품에 대한 이해 (CDB 구성 및 CDB 모니터링을 통한 문제점 확인)
- Data Analytics 상품에 대한 이해 (Cloud Hadoop, Cloud Search)
- Security 상품에 대한 이해 (Certificate Manager 포함)
- AI / Application 상품에 대한 이해 (NAVER AI 서비스)
- CI/CD 상품에 대한 이해 (Cloud Function, SourceCommit, SourceBuild, SourceDeploy, SourcePipeline, **SourceBand**)
- 네이버 클라우드 플랫폼 아키텍처 사례 연구

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

## ☁️ 클라우드 컴퓨팅이란 무엇인가

민준이 NCP 공부를 시작하면서 가장 먼저 마주친 질문은 단순했다. "클라우드가 정확히 뭔가요?"

**클라우드란 인터넷 기반의 컴퓨팅**을 의미하며, 서버·네트워크·스토리지·애플리케이션·서비스 등 다양한 컴퓨팅 리소스에 장소나 시간 제약 없이 쉽고 빠르게 접근할 수 있게 해줍니다.

클라우드에서는 다음과 같은 것들을 서비스로 제공합니다:

- **인프라 서비스**: 서버, 스토리지, 네트워킹, 데이터베이스
- **플랫폼 서비스**: 관리형 데이터베이스, 대용량 데이터 분석, 미디어 처리
- **애플리케이션 서비스**: AI 기능, 개발 도구, 비즈니스 솔루션
- **보안·관리 도구**: 다양한 관리 도구와 보안 기능

이러한 서비스는 **온디맨드(On-Demand)**로 필요할 때 직접 생성해 수 분 만에 사용할 수 있으며, **종량과금제**가 적용됩니다.

### 클라우드의 5가지 장점

| 장점 | 설명 |
|------|------|
| **시간 절약** | 하드웨어/소프트웨어를 직접 구매·구축할 필요 없이 대여하여 사용 |
| **인력·시간 자유** | 완성형 서비스를 바로 가져다 쓰므로 개발 인력과 시간 제약 없음 |
| **생산성 향상** | 인터넷만 되면 언제 어디서나 접속 가능 |
| **유연한 확장** | 필요한 IT 자원을 수 분 내로 생성, 규모 변경이 용이 |
| **CAPEX → OPEX 전환** | 선투자 없이 사용한 만큼 비용 지불 |

> **CAPEX(Capital Expenditure)**: 미래의 이윤을 창출하기 위해 지출된 비용 (서버 구매 등)  
> **OPEX(Operating Expenditure)**: 운영 비용으로 이미 갖춰진 인프라를 운영하는 데 드는 비용  
> **클라우드는 CAPEX를 OPEX로 대체하는 효과**를 가져다줍니다.

클라우드 사업자는 **규모의 경제** 덕분에 소규모 기업이 혼자 구축할 때보다 훨씬 저렴하고 안정적인 인프라를 제공할 수 있습니다. 또한 지속적인 기술 혁신을 통해 새로운 서비스를 꾸준히 제공하며, 인프라 유지 관리 부담을 없애 사업의 핵심에 집중할 수 있도록 해줍니다.

### 클라우드 서비스 유형

| 유형 | 풀네임 | 설명 |
|------|--------|------|
| **IaaS** | Infrastructure as a Service | Computing, Networking, Storage 등 IT 리소스를 서비스로 사용 |
| **CaaS** | Container as a Service | OS 레벨이 아닌 컨테이너 레벨의 기능을 서비스로 사용 |
| **PaaS / aPaaS** | Platform as a Service | 기본 인프라를 직접 관리하지 않으면서 플랫폼 소프트웨어를 서비스로 사용 |
| **FaaS** | Function as a Service | 컴퓨팅 리소스를 기능(function code) 단위로 실행, 서버 관리 없이 앱 수행 |
| **SaaS** | Software as a Service | 완제품 형태로 제공되는 소프트웨어·애플리케이션·솔루션을 서비스로 사용 |

### 클라우드 배포 형태

- **Public Cloud**: 클라우드 사업자가 운영하는 공용 클라우드 (NCP가 여기에 해당)
- **Private Cloud**: 기업 전용 사설 클라우드
- **Hybrid Cloud**: Private Cloud + Public Cloud를 결합하여 이용하는 형태 (중요한 정보는 내부, 덜 중요한 정보는 외부 클라우드에 보관)
- **Multi Cloud**: 여러 Public Cloud를 동시에 사용하는 형태 (A 클라우드 장애 시 B 클라우드로 전환)

---

## 🏢 NAVER Cloud Platform 소개

### NCP는 어디서 왔는가 — 실제 서비스에서 검증된 인프라

NCP는 단순히 새로 만든 클라우드 서비스가 아닙니다. 다음 규모의 서비스를 실제로 운영하면서 쌓은 기술력이 그대로 NCP에 담겨 있습니다:

- **NAVER**: 국내 최대 인터넷 서비스, **4,200만 회원**, 모바일 하루 순방문자 **3,000만**
- **LINE**: 월간 **2억 이상**의 액티브 사용자, 하루 **170억 개**의 메시지를 처리하는 글로벌 모바일 서비스 (전 세계 230개 국가)

즉, 국내 최대·글로벌 최상위 서비스에 사용된 클라우드 인프라와 **동일한 기술**로 구현되었습니다.

### NCP 역사 (History)

| 연도 | 주요 사건 |
|------|-----------|
| **2012년** | Ncloud v1.0 자체 개발 (네이버 내부 핵심 인프라를 클라우드로 전환 시작), 클라우드 서비스 상생지원 프로그램 '에코스퀘어' 시작 |
| **2017년 4월** | NAVER Cloud Platform 정식 출시 (기존 Ncloud v2.0을 본격적인 퍼블릭 클라우드로 제공), 출시 당시 상품 수 **22개**, 리전 **1개(KR)** |
| **2017년 7월** | 공공 클라우드 출시 |
| **2017년 12월** | 상품 수 **80개**, 글로벌 리전 **6개**(KR, US-W, SG, HK, JP, DE)로 성장 — 한 해에 매월 5~6개 상품 출시 |
| **2017년** | ISMS, PIMS, CSAP 인증 취득, CSA STAR Gold 등급 취득 |
| **2019년** | 국내 최초로 금융/핀테크 기업 전용 **금융 클라우드** 출시 (금융 컴플라이언스 준수 접근 방식 최소화) |
| **2021년 이후** | 각 컴플라이언스에 맞는 클라우드 제공: **공공 / 금융 / 민간** 클라우드 |

### NCP의 6가지 핵심 강점

1. **세계적으로 인정받은 보안 기술** — 서비스 안정성과 정보보호 관련 국제/국내 인증 다수 취득, 단 1건의 보안 사고도 허용하지 않는 엄격한 보안
2. **다양한 서비스 플랫폼 기술** — 네이버와 LINE을 비롯한 다양한 분야의 서비스 운영 경험으로 축적한 플랫폼 기술
3. **국내 최대 IT 서비스의 안정적인 인프라** — 정전과 지진에 대비해 튼튼하게 지어진 자체 데이터 센터, 3번 백업한 데이터를 분산 저장하는 스토리지 기술
4. **쉽고 간편한 웹 기반 관리 도구** — 누구나 쉽게 서비스를 사용하고 관리할 수 있도록 간편하게 설계된 웹 콘솔
5. **글로벌 메이저급 인프라 품질** — 글로벌 데이터 센터 간 최적 속도를 낼 수 있는 전용 회선, 해외 주요 통신사와의 협력
6. **24시간 365일 사용자 지원** — 분야별 전문 기술진으로 구성된 사용자 센터 상시 운영

### NCP 클라우드 관리 콘솔

NCP의 관리 콘솔은 **100여 가지 서비스 및 상품**을 사용할 수 있는 웹 기반 사용자 인터페이스입니다.

- **MC(Main Console)**: 전반적인 이용 상품 내용, 공지사항, 신규 상품 출시 내용 확인
- 상품별 상세 사용 가이드 문서 연결
- 자주 사용하는 상품 고정하기
- 사용할 리전 선택
- 상품 소개, 이용 관리, 계정 관리, 문의하기 등은 포털 페이지에서 제공

---

## 🌏 리전과 멀티존 구성

민준은 클라우드빵집을 전국 배달로 시작했지만, 언젠가 해외 진출도 꿈꾸고 있었다. 그래서 리전 구조를 꼼꼼히 살펴봤다.

### 글로벌 리전

NCP는 다양한 글로벌 리전을 제공합니다. 2017년 출시 당시 한국(KR) 단일 리전에서 시작해, 같은 해 말에 이미 6개 리전(KR, US-W, SG, HK, JP, DE)으로 확장했습니다.

### 멀티존(Multi-Zone) 구성

**존(Zone)** 이란 국가 단위의 리전 내에 물리적으로 분리되어 존재하는 데이터 센터 및 네트워크를 의미합니다.

멀티존을 지원하는 리전은 다음 3곳뿐입니다:

> **한국(KR), 싱가폴(SG), 일본(JP) 리전만 멀티존 구성 제공**

멀티존을 활용하면 데이터 센터 간 이중화 구성이 가능합니다:

- 로드 밸런서를 이용한 이중화 구성
- 멀티존 간 동일 IP 대역 활용
- 백업 및 DR(Disaster Recovery) 서비스 활용

---

## 🛡️ 보안 인증 현황

NCP는 단순한 기술 주장이 아닌, 공인된 보안 인증으로 신뢰성을 증명합니다.

| 인증 | 설명 |
|------|------|
| **ISO 27001** | 글로벌 정보보호 관리체계 인증 |
| **SOC 2·3** | 글로벌 서비스 통제 수준 표준 인증 |
| **ISO 27017** | 글로벌 클라우드 보안 표준 인증 |
| **PIMS** | 개인정보보호 관리체계 인증 |
| **ISO 27018** | 글로벌 클라우드 개인정보보호 표준 인증 |
| **ISMS** | 정보보호 관리체계 인증 |
| **CSA STAR** | 글로벌 클라우드 통제 수준 표준 인증 (국내 최초 취득) |
| **CSAP** | 클라우드 컴퓨팅 서비스 보안 인증 (공공 클라우드 보안 인증) |
| **PCI DSS** | 지불 카드 산업 정보 보호 표준 인증 |

보안 장비로는 **IDS, DDoS 방어, IPS, WAF, Anti-Virus** 등이 기본 제공됩니다.

특히 공공 클라우드의 경우, **국내 CC 인증**을 받은 하드웨어 및 보안 장비를 사용하는 강화된 보안 체계를 적용하여 한국인터넷진흥원(KISA)으로부터 '클라우드 서비스 보안 인증(CSAP)'을 획득했습니다.

---

## 📦 NCP 전체 상품군 개요

민준은 콘솔에 처음 접속했을 때, 메뉴의 방대함에 압도되었다. 도대체 어떤 상품이 얼마나 있는 건지. 교재에서는 상품을 크게 5개 카테고리로 나누어 설명합니다.

### 인프라 상품군

**Compute(컴퓨팅)**
- Server: 클라우드에서 VM 서버 제공
- Auto Scaling: VM 수를 사용량에 따라 조절
- Cloud Function: 서버 생성 없이 코드 로직 실행

**Storage(스토리지)**
- Object Storage: 대량 파일 저장하는 오브젝트 스토리지
- NAS: 여러 서버가 파일을 공유하는 스토리지
- Backup: VM 내의 파일이나 DB를 백업
- Archive Storage: 뛰어난 확장성과 가용성, 데이터 내구성을 보장하는 오브젝트 스토리지
- Object Migration / Data Teleporter: 다른 클라우드 환경의 데이터를 네이버 클라우드로 안전하고 빠르게 마이그레이션

**Containers(컨테이너)**
- Container Registry: 컨테이너 이미지 저장/관리
- Kubernetes Service: 컨테이너 오케스트레이션 툴
- Mirror Source / Mirror Destination: Classic 플랫폼 마이그레이션 지원

**Networking(네트워크)**
- Load Balancer: 부하 분산을 위한 L4
- VPC: 가상의 사설망 구성
- Classic Path: Classic 플랫폼에서 VPC 서비스 연결
- IPsec VPN: 외부 네트워크와 네이버 클라우드 플랫폼을 사설 네트워크로 연결
- Cloud Connect: 내부 사설망 연결
- DNS / GDNS: DNS 서비스, Global Traffic Manager
- Content Delivery / Global CDN / CDN+: 여러 사용자에게 콘텐츠를 빠르고 안전하게 전달하는 CDN 서비스
- Global Edge: 대규모 글로벌 네트워크로 전 세계 콘텐츠 제공

**Hybrid & Private Cloud**
- NeuroCloud: 네이버 클라우드의 플랫폼을 이용한 고객 전용 사설 클라우드

### 플랫폼 상품군

**Database**
- Cloud DB for MySQL: HA 구성, 모니터링, 백업 등이 지원되는 관리형 MySQL
- Cloud DB for MSSQL: 관리형 MSSQL
- Cloud DB for Cache: 고성능 오픈 소스 In-memory 데이터 스토어 및 캐시
- Cloud DB for MongoDB: 관리형 MongoDB
- Cloud DB for PostgreSQL: 완전 관리형 PostgreSQL
- Database Migration Service: 데이터베이스를 클라우드 환경으로 마이그레이션

**BigData & Analytics**
- Cloud Search: 엘라스틱 서치를 검색 기능을 쉽고 빠르게 구현할 수 있는 Search Engine Service
- Cloud Hadoop: Apache Hadoop, Spark, Hive, HBase 등 오픈 소스 기반 완전 관리형 클라우드 분석 서비스
- Cloud Data Streaming (CDSS): Apache Kafka 클러스터를 제공, 빅데이터 분석에서 머신러닝까지 분석 가능
- Data Flow: 데이터를 쉽게 탐색·변환·이동할 수 있는 ETL/Pipeline 서비스
- Data Stream: 서버리스 완전 관리형 서버리스 메시지 큐 서비스
- Data Query: 서버리스 대화형, 간편하게 데이터를 직접 분석할 수 있는 쿼리 서비스
- Data Catalog: 데이터 자산의 활용성을 강화하는 클라우드 기반의 메타데이터 통합 및 관리 서비스
- NIMORO: 데이터 분석 및 비즈니스 인텔리전스 플랫폼

**Media**
- VOD Station: VOD 스트리밍 서비스
- Image Optimizer: 원본 이미지를 리사이징·실시간 변환 및 전송
- Live Station: 동영상 스트림 실시간 변환 서비스
- Video Player Enhancement: 다양한 디바이스 환경에서 비디오 재생 테스트·배포·최적화 진행 가능한 SaaS 기반 비디오 플레이어 서비스
- Media Connect Center: 통합된 클라우드 상품들을 하나의 환경으로 제공하는 미디어 플랫폼
- PRISM Live Studio: 세계적인 수준의 B2B 실시간 방송 서비스
- Understanding Media AI: 영상 분석 서비스, 미디어 AI 검색
- Multi DRM: 콘텐츠의 저작권을 보호해주는 멀티 DRM 서비스

**Gaming**
- Gamepot: 게임에 필요한 기능을 손쉽게 서비스

**Blockchain**
- Blockchain Service: Hyperledger Fabric을 이용하여 블록체인 프라이빗·컨소시엄 방식의 네트워크를 쉽고 빠르게 구현

### 애플리케이션 & Developer Tools 상품군

**AI Service**
- CLOVA Speech: 정형화되지 않은 말소리를 인식하여 텍스트로 바꿔주는 음성 인식 서비스
- CLOVA OCR: 인쇄물상의 문자를 인식해 디지털 데이터로 자동 추출
- CLOVA Voice / CLOVA Premium Voice: AI 기반 음성 합성 (TTS), 다양하고 자연스러운 음성
- CLOVA Chatbot: 챗봇 대화 모델 작성과 채널 연계
- CLOVA Dubbing: AI 보이스를 입히는 기능, 기술로 제작한 동영상에 다양한 음성 합성
- CLOVA Face Recognition (CFR): 이미지를 판독하고 얼굴 인식
- CLOVA Green Eye: 유해 이미지를 탐지하는 서비스
- CLOVA Studio (HyperCLOVA X 기반): 초대규모 AI, No Code AI 도구
- CLOVA ITEMS: 사용자별 이력을 분석해 관심사와 취향에 맞는 상품을 추천
- Papago Translation: AI 기반 번역
- TensorFlow Cluster / TensorFlow Server: 높은 성능을 가진 컴퓨팅 파워
- CLOVA AiCall: AI 기반 통화 서비스

**Developer Tools (CI/CD)**
- SourceCommit: 보안 검사 기능이 강화된 Git Repository
- SourceBuild: 빌드 서버 운영이 필요 없는 완전 관리형 병렬 빌드 서비스
- SourceDeploy: 배포 프로세스를 자동화하는 자동화 배포 서비스
- SourcePipeline: 리파지토리, 빌드, 배포 프로세스를 통합하여 빠르고 안정적인 S/W 출시를 지원하는 자동화 서비스
- SourceBand: 팀원들과 편리하게 작업을 공유하고 관리할 수 있으며, 직관적인 인터페이스를 통해 작업 일정 및 흐름을 효과적으로 파악할 수 있는 협업 서비스

**Application Service**
- API Gateway: API 서비스 등록 관리
- Outbound Mailer: 대량 메일 전송 서비스
- Geolocation: 사용자 IP에 대한 위치 정보(등 단위 주소) 제공
- Maps: 네이버 지도 API 서비스
- SENS: SMS, Push 메시지 전송 서비스
- RabbitMQ: AMQP 기반의 RabbitMQ 클러스터
- Digital Twin: 실내외 디지털 트윈을 구축해 AR·로봇·스마트 빌딩 등에 활용

**Business Application**
- NAVER Works: S/W 메시지, 메일, 캘린더 등 모바일 업무용 협업 툴
- Ncloud Chat: 모든 비즈니스에서 사용 가능한 Business 전용 채팅 솔루션

### 보안 상품군

| 상품 | 설명 |
|------|------|
| **Basic Security** | 기본으로 모든 고객에게 제공되는 무료 보안 서비스 |
| **Security Monitoring** | IDS, Linux DDoS 백신 방어, WAF, IPS 등 |
| **KMS** | 데이터 암호화 키 생성·관리·보관 |
| **Secret Manager** | 보안 암호를 교체·관리할 수 있는 서비스 |
| **App Safer** | 모바일 탐지·맵 위변조 방지 |
| **File Safer** | 파일의 악성 여부 탐지 |
| **Web Shell Behavior Detector** | 웹쉘 공격 실시간 탐지, 알람 전송 |
| **SSL VPN** | 접속용 SSL, PC agent 방식 SSL |
| **Certificate Manager** | 효율적인 인증서 등록 관리 |
| **Private CA** | 사설 관리 운용 인증서 발급 |
| **Web Security Checker** | 웹 사이트의 취약점 점검 및 리포트 |
| **System Security Checker** | 서버 보안 설정 점검 및 리포트 |
| **Cloud Security Watcher** | 보안 리소스 감시, 현황 식별, 자산 변경 형상 관리 서비스 |
| **Compliance Guide** | 다양한 보안 인증 및 가이드 문서 제공, 프레임워크 준수 현황 점검 |

### 관리/보안(Management & Governance) 상품군

| 상품 | 설명 |
|------|------|
| **Sub Account** | 하위 계정 권한 관리 |
| **Resource Manager** | 다양한 리소스를 논리적 그룹으로 관리 |
| **CLA** | 여러 서비스에서 발생하는 다양한 로그들을 한 곳에 모아 저장하고 손쉽게 분석 |
| **Cloud Insight (Monitoring)** | 클라우드 성능/운영 지표 통합 모니터링, 알람 발송 |
| **Cloud Advisor** | 네이버 클라우드 플랫폼 자체 운영 노하우를 바탕으로 카테고리별 점검 리포트 제공 |
| **Network Traffic Monitoring** | 패킷 정보 수집을 통한 네트워크 트래픽 모니터링 |
| **WMS** | 웹 글로벌 응답 속도 측정, 정상 동작 여부 모니터링 |
| **API Workflow** | HTTP 기반 프로세스 구성, 서비스들의 실행 순서를 구현하는 자동화 |
| **ELSA** | 애플리케이션 서비스 운영 시 발생하는 로그를 빠르게 저장하고 분석 |
| **Cloud Activity Tracer** | 사용자의 작업 내용을 기록하는 로그 상품 |
| **Pinpoint** | 마이크로서비스 아키텍처 성능 개선을 위한 분산 추적 플랫폼 |
| **Service Quota** | 서비스별 한도 및 사용량을 확인하고, 필요 시 한도를 상향할 수 있는 서비스 |
| **Cost Explorer** | 클라우드 비용을 상세 분석하여 최적화 |
| **Organization** | 모든 계정의 결제 및 비용을 중앙에서 통합 확인 |
| **Single Sign-On** | 네이버 플랫폼 계정을 통해 조직 내 클라우드 플랫폼 접근 권한을 통합 관리 |
| **Ncloud Tools & SDK** | 네이버 클라우드 플랫폼과 연동 가능한 다양한 도구, 편리하고 효율적인 관리 및 운영 |

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
시험 합격!
```

---

*다음 챕터: Chapter 1 — 서버를 처음 만드는 순간, 하이퍼바이저를 선택하라*
