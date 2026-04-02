---
title: "최종 총정리 — 시험 직전 체크리스트"
date: 2026-04-02 12:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, final-review, checklist, exam-summary]
description: "NCP NCE 자격증 대비 최종 요약 - 시험 직전 핵심 수치와 개념 체크리스트 (AI·DevOps·아키텍처 포함 전 챕터)"
---
> 이 문서는 NCE 시험 직전, 모든 핵심 수치와 개념을 빠르게 복습하기 위한 최종 정리입니다.

---

## ⚡ Part 1: 컴퓨팅 핵심 수치

### 서버 세대 & 하이퍼바이저

| 세대 | 하이퍼바이저 | AMD 지원 | 지원 환경 |
|------|------------|---------|---------|
| G1/G2 | **XEN** | ❌ | Classic/VPC |
| G3 | **KVM** | ✅ | **VPC만** |

- XEN ↔ KVM 간 **호환 불가**
- CPU: Over Commit **허용** / Memory: Over Commit **불허**

### OS 영역 스토리지

| 하이퍼바이저 | Linux | Windows | 확장 |
|------------|-------|---------|------|
| XEN | **50GB 고정** | **100GB 고정** | **불가** |
| KVM | 최소 **10GB** | 최소 **30GB** | **정지 후 가능** |

### 서버 타입 비율

| 타입 | 비율 | 용도 |
|------|------|------|
| Micro | **1:1** | 테스트, **1년 무료** |
| Standard | **1:4** | 일반 웹/DB |
| High Memory | **1:8** | 고성능 DB |
| High CPU | **1:2** | 과학적 모델링 |
| CPU-Intensive | **1:2** | 딥러닝, 고성능 웹 |

### 특수 서버

| 항목 | Bare Metal | GPU |
|------|-----------|-----|
| 하이퍼바이저 | **없음** | 있음 (Pass Through) |
| GPU 방식 | - | **Pass Through** (GRID 아님!) |
| T4 최대 | - | **2장** |
| V100 최대 | - | **4장** |
| A100 최대 | - | **8장 (Bare Metal만)** |
| 스토리지 추가 | **불가** | 가능 |
| 오토스케일링 | **불가** | **불가** |
| RAID 1+0 | DB/안정성 | - |
| RAID 5 | 일반 웹 | - |

### 서버 Operation 상태 요건

| 작업 | 필요 상태 |
|------|---------|
| 스펙 변경 | **정지** |
| 이미지 생성 | **운영 중/정지 모두** ← 함정! |
| 스토리지 용량 변경 | **정지 또는 연결 해제** |
| 인증키 변경 | **정지** |

### 스토리지 추가 제한

| 하이퍼바이저 | 최대 크기 | 최대 개수 |
|------------|---------|---------|
| XEN | **2TB** | **15개** |
| KVM | **16TB** | **20개** |

### NIC 제한

- 서버당 NIC 최대: **3개**
- NIC당 Secondary IP: **5개**
- NIC당 총 IP: **6개** (Primary 1 + Secondary 5)

### CLI 도구 (함정 주의!)

| 서비스 | CLI |
|--------|-----|
| 일반 서버 | **ncloud CLI** |
| Object Storage | **AWS CLI** |
| Archive Storage | **OpenStack swift** |

---

## ⚡ Part 2: 보안 핵심 비교

### NACL vs ACG

| 항목 | **NACL** | **ACG** |
|------|---------|---------|
| 적용 단위 | **Subnet** | **서버 NIC** |
| 규칙 종류 | Allow + **Deny** | **Allow만** |
| 기본 정책 | **허용** | **차단** |
| 상태 | **Stateless** | **Stateful** |
| 처리 방식 | **우선순위(Rule Number)** | **전체 규칙 평가** |

### ACG 제한

- VPC당: **500개**
- NIC당 적용: **3개**
- 규칙 수: In/Out 각 **50개**

### 포트 번호

| 포트 | 프로토콜 |
|------|---------|
| **21** | FTP |
| **22** | SSH |
| **80** | HTTP |
| **443** | HTTPS |
| **3389** | RDP (Windows) |

---

## ⚡ Part 3: 스토리지 핵심 수치

### 스토리지 3대장

| 항목 | Block | File | Object |
|------|-------|------|--------|
| 접근 | iSCSI/NVMe | **NFS/SMB** | **HTTP REST** |
| 공유 | 1대만 | **여러 서버** | 무제한 |
| 용도 | OS/DB | 파일 공유 | 미디어/CDN |

### Object Storage 제한

| 항목 | 수치 |
|------|------|
| 버킷 최대 | **1,000개** |
| 객체명 최대 | **1,024 Byte** |
| 콘솔 업로드 | **1GB** |
| API 업로드 | **10TB** |
| 콘솔 다운로드 | **2GB** |
| API 다운로드 | **제한 없음** |
| 버킷 용량 | **제한 없음** |

### Archive Storage

- 저장: **싸다** / API: **비싸다**
- 최소 보관 기간: **없음**
- CLI: **OpenStack swift**
- API: S3 + **swift 모두 지원**

### 멀티파트 업로드

- Segment 최대 크기: **5GB**
- 5GB 초과 → **반드시 분할**

### NAS

| 항목 | 수치 |
|------|------|
| 최소 용량 | **500GB** |
| 최대 용량 | **20TB** |
| 증설 단위 | **100GB** |
| Linux 프로토콜 | **NFS** |
| Windows 프로토콜 | **CIFS** |

### 스냅샷 vs 서버 이미지

| 항목 | 서버 이미지 | 스냅샷 |
|------|-----------|--------|
| 단위 | **서버** | **스토리지** |
| 복원 | 서버로 복원 | **추가 스토리지로만** |

### Backup

| 항목 | 수치 |
|------|------|
| 최소 보관 | **7일** |
| 최대 보관 | **364일 (52주)** |

### 백업 방식 비교

| 항목 | 전체 | 차등 | 증분 |
|------|-----|------|-----|
| 복원 파일 수 | **1개** | **2개** | 여러 개 |
| 용량 | 최대 | 중간 | **최소** |
| 시간 | 최장 | 중간 | **최단** |
| 복원 난이도 | 쉬움 | 쉬움 | **복잡** |

---

## ⚡ Part 4: 컨테이너 핵심

### VM vs Container

| 항목 | VM | Container |
|------|-----|---------|
| Guest OS | **있음** | **없음** |
| 커널 | 각자 | **Host OS 공유** |
| 무게 | 무거움 | **가벼움** |

### Dockerfile 명령어

| 명령 | 실행 시점 | 역할 |
|------|---------|------|
| FROM | - | 베이스 이미지 |
| WORKDIR | - | 작업 디렉터리 |
| COPY | - | 파일 복사 |
| RUN | **빌드 시** | 명령 실행 |
| EXPOSE | - | 포트 선언 |
| CMD | **시작 시** | 기본 실행 명령 |

### K8s 컴포넌트

**Master Node:**
- **API Server**: 모든 요청 통로
- **Controller-Manager**: 상태 관리
- **Scheduler**: 노드 배치 결정
- **etcd**: 데이터 저장소

**Worker Node:**
- **kubelet**: Pod 관리 에이전트
- **kube-proxy**: 네트워크/LB 규칙
- **Container Runtime**: 컨테이너 실행

### K8s 오브젝트 계층

```
Deployment > ReplicaSet > Pod > Container
```

- **DaemonSet**: 모든 노드마다 Pod 1개씩

### Service 타입

| 타입 | 외부 접근 | 특징 |
|------|---------|-----|
| **ClusterIP** | ❌ | 내부 통신만 |
| **NodePort** | ✅ | 30000~32767 포트 |
| **LoadBalancer** | ✅ | 클라우드 LB 자동 연동 |

- **Ingress**: L7 라우팅, 뒤에 **NodePort/LoadBalancer** 필요

### NKS

- **Master Node = NCP 관리**
- 사용자: 앱 배포, 모니터링 도구
- Container Registry = **Object Storage 기반** + **CVE 분석**

---

## ⚡ Part 5: 네트워크 핵심 수치

### OSI 7계층 vs TCP/IP 4계층

| OSI | TCP/IP | 데이터 단위 | 주요 프로토콜 |
|-----|--------|-----------|------------|
| 7 응용 | 응용 | Data | HTTP, DNS |
| 4 전송 | 전송 | **Segment** | **TCP, UDP** |
| 3 네트워크 | 인터넷 | **Packet** | **IP** |
| 2 데이터링크 | 네트워크접근 | **Frame** | MAC |

### CIDR 계산

- 호스트 수 = **2^(32 - 비트마스크)**
- /24 = 256개, /25 = 128개, /26 = 64개

### VPC 구조

| 항목 | 수치 |
|------|------|
| VPC Peering | **1:1** (다른 계정 가능) |
| Transit VPC | **1:N** (1개만 생성) |

### 로드밸런서 3종 비교

| 항목 | NLB | Network Proxy LB | ALB |
|------|-----|-----------------|-----|
| 프로토콜 | **TCP, UDP** | **TCP, TLS** | **HTTP, HTTPS** |
| 계층 | **L4** | L4 Proxy | **L7** |
| DSR | **✅** | ❌ | ❌ |
| SSL 오프로드 | ❌ | **✅** | **✅** |
| Sticky Session | **✅** | **❌** | **✅** |
| 로깅 | ❌ | ✅ | ✅ |

### DNS 레코드 9종

| 레코드 | 역할 |
|--------|------|
| **A** | 도메인 → IPv4 |
| **AAAA** | 도메인 → IPv6 |
| **CNAME** | 별명 (**다른 레코드와 공존 불가**) |
| **MX** | 메일 서버 (**낮은 Preference = 높은 우선순위**) |
| **SPF** | 스팸 방지 |
| **TXT** | 텍스트 정보 |
| **SRV** | 서비스 위치 |
| **CAA** | 가짜 인증서 방지 |
| **DS** | DNSSEC 서명 |

- TTL 기본: **300초 (5분)** / 최대: **7일**

### CDN & 미디어

| 항목 | 수치 |
|------|------|
| Live Station 최대 해상도 | **1920×1080@60fps** |
| Live Station 동시 변환 | **5개** |
| Live Station 최대 비트레이트 | **20Mbps** |
| Live Station CDN 동시 접속 | **1만 명** |
| Live Station Input | **RTMP** |
| Live Station Output | **HLS, LL-HLS, MPEG-DASH** |

---

## ⚡ Part 6: 데이터베이스 핵심 수치

### DB 종합 비교표

| DB | 포트 | 최대 스토리지 | 최대 Slave | 백업 보관 |
|----|------|-------------|-----------|---------|
| **MySQL** | **3306** | **6TB** | **10대** | 30일 |
| **MSSQL** | **1433** | **2TB** | **5대** | 30일 |
| **PostgreSQL** | **5432** | **6TB** | **5대** | 30일 |
| **Redis** | **6379** | - (메모리) | 샤드당 **4대** | **7일** |
| **MongoDB** | **17017** | **2TB** | **7대** | 30일 |

### MySQL 핵심 설정

| 파라미터 | 기본값 |
|---------|--------|
| `max_connections` | **3000** |
| `wait_timeout` | **28800초** |
| `character_set_server` | **utf8mb4** |

### MySQL Fail-over

- 별도 모니터 서버, **1분 단위** 체크
- 업그레이드 순서: **Recovery → Slave → Master**
- Read Replica 최대: **10대**

### MSSQL Slave

- **Log Shipping** 방식 (실시간 X)
- **BI/Batch 전용** (실시간 읽기 불가)
- 읽기 가능 시간: 하루 최대 **20시간**
- 최대 Slave: **5대**

### Redis Cluster

- Shard: **3 ~ 10개**
- Shard당 Replica: 최대 **4개**
- 백업 보관: **7일** (다른 DB는 30일!)

---

## ⚡ Part 7: 빅데이터 핵심 수치

### Cloud Search

- 기본 컨테이너: **2개** (Indexer 1 + Searcher 1)
- 지원 언어: **7개**

### Cloud Hadoop

- Master Node: 기본 **2개** (HA)
- 주 데이터 소스: **Object Storage**
- Edge Node 접속: **SSL VPN**

### HDFS

- NameNode: **메타데이터** 관리
- DataNode: 실제 데이터, **3개 복사본**
- Heartbeat 주기: **3초**

### MapReduce

```
Map → Shuffle → Reduce
```

### YARN

- Resource Manager: 전체 자원 관리
- Node Manager: 각 노드 자원 모니터링
- Application Master: 앱별 자원 요청

### Kafka vs RabbitMQ

| 항목 | Kafka | RabbitMQ |
|------|-------|----------|
| 모델 | **Topic** | **Queue** |
| 전달 방식 | **Pull** | **Push** |
| 소비 후 | **유지** | **삭제** |
| 저장 | **파일 시스템** | 메모리 |

### CDSS (관리형 Kafka)

- 최소 노드: **4개** (Manager 1 + Broker 3)
- Broker 수: 감소 **불가**

---

## ⚡ Part 8: 보안 핵심

### 보안 장비 역할

| 장비 | 역할 |
|------|------|
| **Firewall** | IP/Port 패킷 필터 (L3/L4) |
| **IDS** | 탐지만 (차단 X) |
| **IPS** | 탐지 + 차단 |
| **WAF** | 웹 공격 차단 (L7) |

### System Security 파일 권한

| 파일 | 권한 |
|------|------|
| `/etc/shadow` | **400** |
| `/etc/passwd` | **644** |

- Windows 인증: **NTLMv2**
- UMASK: 022 또는 027

### KMS 봉투 암호화

```
Root Key (NCP) → Master Key (고객) → Data Key → 데이터
```

### Private CA

- 최대 CA: **10개**
- 최대 인증서: **30,000개**

### Webshell Behavior Detector

- **Agent** 기반 설치
- **행위(Behavior) 기반** 탐지

---

## ⚡ Part 9: AI 서비스 핵심 수치

### CLOVA Chatbot

| 항목 | 내용 |
|------|------|
| 지원 언어 | **6개** (한/일/영/중/태/인도네시아) |
| 네거티브 최대 | **20개** |
| 대화 우선순위 | **객관식 버튼 > 컨텍스트 > 일반 대화** |

### CLOVA OCR 모델 비교

| 항목 | Basic | Premium |
|------|-------|---------|
| 인식 대상 | 활자체만 | 활자체 + **필기체** |
| 멀티박스 | ❌ | **✅** |
| 체크박스 | ❌ | **✅** |
| 필드 유형 | ❌ | **✅** |

- TEXT OCR 경로: **`/general`**
- Template OCR 경로: **`/infer`**

### CLOVA Speech (STT)

- API 방식: **object-storage / url / upload** 3가지
- 키워드 부스팅 최대: **500건**
- 부스팅 불가: **1음절** 단어 (오인식 위험)
- 부스팅 가능 문자: **한글, 영어, 숫자**만
- 기본 언어: **ko-KR** / 기본 방식: **async**

### CLOVA Voice (TTS)

- 음성 옵션: **100가지** (성별·연령 포괄)
- Standard/Premium 모두 **실시간 API 호출 불가**
- Standard/Premium 모두 **재가공/재편집 불가**

### Papago

- 언어 감지 지원: **12개 언어**
- 높임말 번역: 한국어 전용 추가 옵션

### Search Trend

- 키워드 그룹: 최대 **5개**
- 그룹당 검색어: 최대 **20개**
- 데이터 제공: **2016.01.01**부터

---

## ⚡ Part 10: DevOps Source 핵심

### Source 4총사 역할

```
SourceCommit  → 프라이빗 Git 리포지토리
SourceBuild   → 자동 빌드
SourceDeploy  → 자동 배포
SourcePipeline → 3개 통합 CI/CD 파이프라인
```

### SourceCommit 권한 3종

| 권한 | 가능 작업 | 비고 |
|------|---------|------|
| **ADMIN** | CLONE/PULL/PUSH + 설정변경/삭제 | 생성자에게 **자동 부여** |
| **WRITE** | CLONE/PULL/PUSH | - |
| **READ** | CLONE/PULL만 | - |

### SourceBuild 제약

- 빌드 OS: **Ubuntu 16.04** 만 지원
- 빌드 소스: SourceCommit + **GitHub** (외부도 가능)
- 빌드 런타임: base / java / .NET / android-java / python / nodejs
- 서브계정 전체 권한: `NCP_INFRA_MANAGER`
- 서브계정 프로젝트 생성: `NCP_SOURCE_BUILD_MANGER`

### SourceDeploy 지원 환경

| 배포 타겟 | 지원 |
|---------|------|
| NCP Server | ✅ |
| Auto Scaling | ✅ |
| Kubernetes Service | ✅ |
| Object Storage | ✅ |

- 지원 OS: **CentOS, Ubuntu**
- 서브계정 권한: `NCP_SOURCE_DEPLOY_MANGER`

### SourcePipeline 구성

```
Source Phase (SourceCommit)
     ↓
Build Phase (SourceBuild)
     ↓
Deploy Phase (SourceDeploy)
```

- **3개 서비스 모두** 이용해야 사용 가능
- **병렬 실행** 지원

---

## ⚡ Part 11: 클라우드 아키텍처 & 서버리스

### 아키텍처 마이그레이션 3단계

```
Lift & Shift → Cloud-optimized → Cloud-native
```

### 3-tier 계층

| 계층 | 역할 |
|------|------|
| Presentation | 사용자 인터페이스 (웹 서버) |
| Business Logic | 핵심 비즈니스 처리 (앱 서버) |
| Data | 데이터 저장 (DB 서버) |

### 모놀리식 vs 마이크로서비스

| 항목 | 모놀리식 | 마이크로서비스 |
|------|---------|-------------|
| 배포 | 전체 배포 | **독립 배포** |
| 확장 | 전체 확장 | **서비스별 확장** |

- Scale-out/in이 작동하려면 **N-tier Architecture** 사전 설계 필요

### Cloud Functions (서버리스)

- Action, Trigger 모두 **Entity**로 통칭
- Entity 이름: **중복 불가**
- 지원 언어: **Node.js 6/8, Python 3, Java 8, Swift 3, PHP 7, .Net, Go** (7종)
- 코드 업로드: 콘솔 작성 or **ZIP(Java는 JAR)** 파일

### Cloud IoT Core

- 프로토콜: **MQTT** (경량 메시지 프로토콜)
- 인증: **X.509 인증서** + **TLS** 암호화 통신

---

## 🎯 자주 나오는 함정 문제 TOP 28

1. "서버 이미지 생성은 정지 상태에서만 가능하다?" → **X (운영 중에도 가능)**

2. "Object Storage CLI는 ncloud CLI다?" → **X (AWS CLI)**

3. "Archive Storage는 최소 보관 기간이 있다?" → **X (없음)**

4. "스냅샷으로 서버를 바로 만들 수 있다?" → **X (추가 스토리지로만)**

5. "GPU 서버는 NVIDIA GRID 방식이다?" → **X (Pass Through)**

6. "ACG는 Deny 규칙을 설정할 수 있다?" → **X (Allow만)**

7. "NACL은 Stateful이다?" → **X (Stateless)**

8. "KVM OS 영역은 운영 중에도 확장 가능하다?" → **X (서버 정지 후)**

9. "Ingress 뒤에 ClusterIP Service를 연결해도 된다?" → **X (NodePort/LB 필요)**

10. "NKS 사용 시 사용자가 Master Node를 관리한다?" → **X (NCP가 관리)**

**네트워크/DB/보안:**

11. "Network Proxy LB는 Sticky Session을 지원한다?" → **X (NLB, ALB만 지원)**

12. "MSSQL Slave는 실시간 읽기 분산에 쓸 수 있다?" → **X (Log Shipping, BI/Batch 전용)**

13. "CNAME은 같은 도메인에 MX 레코드와 공존할 수 있다?" → **X (공존 불가)**

14. "MX Preference 값이 클수록 우선순위가 높다?" → **X (낮을수록 우선순위 높음)**

15. "Redis 백업 보관 기간은 다른 DB와 같이 30일이다?" → **X (Redis만 7일)**

16. "NCP MongoDB 포트는 27017이다?" → **X (NCP는 17017)**

17. "CDSS Broker 수는 줄일 수 있다?" → **X (증가만 가능, 감소 불가)**

18. "IDS는 침입을 탐지하고 차단한다?" → **X (탐지만, 차단은 IPS)**

19. "Live Station은 UDP를 입력 프로토콜로 사용한다?" → **X (RTMP 사용)**

20. "Webshell Detector는 서버리스(에이전트 불필요)다?" → **X (Agent 설치 필요)**

**AI & DevOps:**

21. "CLOVA Chatbot은 한국어만 지원한다?" → **X (6개 언어 지원)**

22. "CLOVA OCR Basic 모델은 필기체를 인식한다?" → **X (활자체만, Premium이 필기체)**

23. "CLOVA Speech API 부스팅은 1음절 단어도 가능하다?" → **X (1음절 불가, 오인식 위험)**

24. "CLOVA Voice Premium 플랜은 실시간 API 호출이 가능하다?" → **X (두 플랜 모두 실시간 API 불가)**

25. "SourceBuild는 CentOS 환경에서 빌드된다?" → **X (Ubuntu 16.04만 지원)**

26. "SourcePipeline은 SourceCommit 없이도 사용할 수 있다?" → **X (3개 서비스 모두 필요)**

27. "Cloud Functions의 Action과 Trigger는 이름이 같아도 된다?" → **X (Entity 이름 중복 불가)**

28. "Cloud IoT Core는 HTTP 프로토콜을 사용한다?" → **X (MQTT 프로토콜)**

---

## 📊 최종 키워드 매핑

```
XEN → G1/G2, OS고정, 스토리지 2TB×15
KVM → G3, OS확장, 스토리지 16TB×20
NACL → Subnet, Stateless, Deny가능, 우선순위
ACG → NIC, Stateful, Allow만, 전체평가
Object Storage → AWS CLI, S3호환, 1GB/10TB
Archive → swift, 저장싸고API비쌈, 최소보관기간없음
NAS → 500GB~20TB, NFS/CIFS, 여러서버
증분백업 → 최소용량/최단시간, 복잡복원
차등백업 → 2개로복원, 누적증가
K8s Master → APIServer/Scheduler/etcd/Controller
K8s Worker → kubelet/kube-proxy/Runtime
Deployment > ReplicaSet > Pod
Service → ClusterIP/NodePort/LoadBalancer
NKS → Master=NCP관리, 앱=사용자
NLB → TCP/UDP, L4, DSR지원, 로깅X
ALB → HTTP/HTTPS, L7, 경로기반, CPS보장
DNS CNAME → 별명, 다른레코드 공존불가
DNS MX → 메일서버, 낮은값=높은우선순위
MySQL → 3306, 6TB, 10Replica, 1분Failover
MSSQL → 1433, 2TB, LogShipping, BI/Batch
PostgreSQL → 5432, 6TB, Slave5대
Redis → 6379, InMemory, Shard3~10, 복제4, 백업7일
MongoDB → 17017, 2TB, Slave7대
Kafka → Topic, Pull, 파일저장, 삭제안됨
RabbitMQ → Queue, Push, 소비후삭제
CDSS → 최소4노드(Manager1+Broker3), 감소불가
Cloud Search → 컨테이너2개, 7개언어
Hadoop → Master2개, ObjectStorage연동
HDFS → NameNode(메타), DataNode(3복사, 3초Heartbeat)
MapReduce → Map→Shuffle→Reduce
IDS → 탐지만 / IPS → 탐지+차단 / WAF → 웹공격L7
KMS → 봉투암호화(Root→Master→Data Key)
Private CA → 10개CA, 30000개인증서
Webshell → Agent설치, 행위기반탐지
Chatbot → 6개언어, NLU+앙상블, 네거티브최대20개
챗봇우선순위 → 객관식버튼 > 컨텍스트 > 일반대화
OCR Basic → 활자체만, Template/General지원
OCR Premium → 활자체+필기체, 멀티박스/체크박스/필드유형
OCR API → /general(TEXT), /infer(Template)
CLOVA Speech → STT, 부스팅최대500건, 1음절불가, 한글영어숫자만
CLOVA Voice → TTS, 100가지음성, 실시간API불가
Papago → 번역+언어감지12개언어+높임말번역
SearchTrend → 그룹5개, 그룹당20개, RESTful API
SourceCommit → 프라이빗Git, ADMIN/WRITE/READ, FileSafer연동
SourceBuild → Ubuntu16.04만, GitHub소스가능, 빌드런타임6종
SourceDeploy → CentOS/Ubuntu, 타겟4종(Server/AutoScaling/NKS/ObjStorage)
SourcePipeline → Source→Build→Deploy, 3서비스모두필요, 병렬실행
CloudFunctions → 서버리스, Action/Trigger=Entity(중복불가), 7개언어
CloudIoTCore → MQTT프로토콜, X.509인증서+TLS
아키텍처 → LiftShift→CloudOptimized→CloudNative
3tier → Presentation/BusinessLogic/Data
ScaleOut → N-tier사전고려, AutoScaling, MultiAZ
```

---

*합격을 응원합니다! 클라우드빵집 CTO처럼 NCP를 자유자재로 다루는 그날까지!* 🍞☁️
