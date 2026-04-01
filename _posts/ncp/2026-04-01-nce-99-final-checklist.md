---
title: "최종 총정리 — 시험 직전 체크리스트"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, final-review, checklist, exam-summary]
description: "NCP NCE 자격증 대비 최종 요약 - 시험 직전 핵심 수치와 개념 체크리스트"
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

## 🎯 자주 나오는 함정 문제 TOP 20

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
```

---

*합격을 응원합니다! 클라우드빵집 CTO처럼 NCP를 자유자재로 다루는 그날까지!* 🍞☁️
