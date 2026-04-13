---
title: "Chapter 11: 주방 총관리자 — Kubernetes (K8s)"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, kubernetes, k8s, pod, deployment, service]
description: "NCP NCE 자격증 대비 학습 자료 - Kubernetes 핵심 컴포넌트와 오브젝트 구조 정리"
---
> **학습 목표**: K8s 컴포넌트(Master/Worker 구분), Pod/ReplicaSet/Deployment 계층, Service 타입 3가지, DaemonSet, Ingress, ConfigMap/Secret, K8s 주요 기능 8가지를 완벽히 구분한다.

---

## 이야기: 컨테이너가 100개가 되면?

클라우드빵집이 폭발적으로 성장했다.
컨테이너가 처음엔 5개, 다음엔 50개, 어느새 100개.

> "이걸 다 수동으로 관리해? 어느 서버에 올리고, 하나 죽으면 다시 살리고..."

개발팀 리더가 말했다.

> "**쿠버네티스(Kubernetes)** 써. 컨테이너 오케스트레이션 도구야.
> 스케줄링, 자동 복구, 스케일링을 알아서 해줘."

---

## 🔑 핵심 개념 1: 컨테이너 오케스트레이션이 없다면?

컨테이너 오케스트레이션 없이 대규모 컨테이너를 관리하면 이런 문제가 생긴다.

**배포 문제**
- 각 서버의 IP를 직접 찾고, 서버에 접속해서 docker 명령어로 컨테이너를 실행/종료해야 한다.
- 새 컨테이너를 실행하려면 빈 서버를 일일이 찾아야 한다.

**서비스 검색 문제**
- 서버가 많아지고 서버 IP가 업데이트되면서 관리자가 모두 추적하는 것이 어려워진다.
- 로드밸런서와 서버 IP 설정을 관리자가 직접 관리해야 한다.

**서비스 노출(Gateway) 문제**
- nginx 같은 외부 프록시 서버를 두고, 들어오는 host 요청에 따라 내부 컨테이너에 연결하는 과정을 수동으로 처리해야 한다.

**모니터링 문제**
- 컨테이너가 갑자기 죽으면 일일이 로그를 보고 서버를 다시 띄워야 한다.
- 트래픽이 많아지면 부하가 걸려 애플리케이션이 느려진다.

이런 문제를 **자동화**해주는 도구 = **컨테이너 오케스트레이션**  
대표 도구: **Kubernetes**, Docker Swarm, Apache MESOS

### 컨테이너 오케스트레이션 주요 기능

| 기능 | 설명 |
|------|------|
| **스케줄링** | 미리 정의된 제약조건 및 우선순위에 따라 사용 가능한 호스트 리소스에 컨테이너 할당 |
| **스케일링** | 애플리케이션 수요에 따라 컨테이너 인스턴스 수를 동적으로 조정 |
| **로드 밸런싱** | 네트워크 트래픽을 컨테이너 간에 균등하게 분산하여 리소스 사용을 최적화하고 성능 향상 |
| **상태 모니터링** | 컨테이너 성능을 모니터링하고 장애가 발생한 컨테이너를 새 컨테이너로 교체 |

---

## 🔑 핵심 개념 2: Kubernetes 컴포넌트 구조 (⭐⭐ 최빈출!)

K8s 클러스터는 **Master Node(Control Plane)** + **Worker Node**로 구성.

```
[Kubernetes 클러스터 구조]

┌─────────────────────────────────────┐
│         Master Node (Control Plane)  │
│  ┌──────────┐  ┌─────────────────┐  │
│  │API Server│  │Controller-Manager│  │
│  └──────────┘  └─────────────────┘  │
│  ┌──────────┐  ┌──────────────────┐ │
│  │Scheduler │  │      etcd        │ │
│  └──────────┘  └──────────────────┘ │
└─────────────────────────────────────┘
          │           │           │
┌─────────▼──┐ ┌──────▼────┐ ┌───▼──────────┐
│ Worker 1   │ │ Worker 2  │ │  Worker 3    │
│ kubelet    │ │ kubelet   │ │  kubelet     │
│ kube-proxy │ │ kube-proxy│ │  kube-proxy  │
│ Container  │ │ Container │ │  Container   │
│ Runtime    │ │ Runtime   │ │  Runtime     │
│ [Pod][Pod] │ │ [Pod]     │ │  [Pod][Pod]  │
└────────────┘ └───────────┘ └─────────────┘
```

**Control Plane(컨트롤 플레인)**: 클러스터의 상태를 유지하고 제어하는 역할.  
클러스터에 관한 전반적인 결정을 수행하고 이벤트를 감지하여 반응한다.

---

### Master Node 컴포넌트

| 컴포넌트 | 역할 |
|---------|------|
| **API Server** | 모든 요청을 처리하는 역할. 사용자와 컨트롤 플레인 간의 통신 창구 |
| **Controller-Manager** | 구성요소 복제, 워커 노드 추적, 노드 장애 처리 등 클러스터 수준 기능 수행 |
| **Scheduler** | 상황에 맞게 적절한 Worker Node를 선택하여 애플리케이션 구성요소에 할당 |
| **etcd** | 클러스터 내의 데이터를 담는 저장소. 클러스터 구성을 지속적으로 저장하는 안정적인 분산 Key-Value DB |

> 📌 **시험 단답형 패턴**  
> "사용자 요청을 제일 먼저 받는 곳?" → **API Server**  
> "어떤 노드에 배포할지 결정하는 곳?" → **Scheduler**  
> "클러스터 상태 데이터를 저장하는 곳?" → **etcd**

---

### Worker Node 컴포넌트

Worker Node는 컨테이너화된 애플리케이션을 동작하고 유지시키는 역할을 한다.

| 컴포넌트 | 역할 |
|---------|------|
| **kubelet** | API Server와 통신하며 Node에 할당된 Pod의 상태를 체크하고 관리하는 에이전트 |
| **kube-proxy** | 애플리케이션 간 네트워크 트래픽을 분산 및 연결 (Pod로 연결되는 네트워크 관리) |
| **Container Runtime** | Pod를 통해 배포된 컨테이너를 실행하는 소프트웨어 (예: Docker, containerd) |

---

## 🔑 핵심 개념 3: K8s 핵심 오브젝트 계층 (⭐⭐ 최빈출!)

```
Deployment (배포 전략 관리)
    └─ ReplicaSet (Pod 개수 유지)
            └─ Pod (최소 배포 단위)
                    └─ Container (실제 앱)
```

### Pod (파드)

- K8s에서 **가장 작고 기본적인 배포 단위**
- 보통 1개의 컨테이너를 담지만, 강하게 결합된 다중 컨테이너도 가능 (사이드카 패턴)
- Pod는 언제든 죽고 새로 생길 수 있어 **IP가 계속 변함** → 직접 IP로 통신 지양

### ReplicaSet (레플리카셋)

- **"지정된 개수의 Pod를 항상 유지"** 하는 역할
- Pod 하나가 죽으면 즉시 새 Pod를 생성하여 replica 수를 복원
- **한계**: 롤링 업데이트(무중단 배포) 기능 없음

### Deployment (디플로이먼트)

- ReplicaSet을 감싸는 상위 개념
- **롤링 업데이트(Rolling Update)** 와 **롤백(Rollback)** 담당
- CPU 사용률 같은 metric을 기반으로 pod의 replicaset을 스케줄링하여 **수평적 확장(Horizontal Scaling)** 가능
- 실제 운영에서는 대부분 Deployment를 사용

```
[버전 업데이트 시 Deployment 동작]
v1 ReplicaSet [Pod][Pod][Pod]
      ↓ 배포 시작
v2 ReplicaSet [Pod] (새로 생성)
v1 ReplicaSet [Pod][Pod] (하나씩 줄어듦)
      ↓ 완료
v2 ReplicaSet [Pod][Pod][Pod]
v1 ReplicaSet (삭제 또는 대기)
→ 서비스 중단 없이 업데이트!
```

---

## 🔑 핵심 개념 4: DaemonSet

**DaemonSet** = 클러스터 내 **모든 노드(또는 특정 노드)마다 Pod 1개씩** 강제 실행.

```
Worker Node 1 → Pod (모니터링 에이전트) 1개
Worker Node 2 → Pod (모니터링 에이전트) 1개
Worker Node 3 → Pod (모니터링 에이전트) 1개
↑ 새 노드 추가 시 자동으로 Pod도 추가됨
```

주요 용도:
- 모니터링 에이전트 (Prometheus Node Exporter)
- 로그 수집기 (Fluentd)
- 네트워크 플러그인

---

## 🔑 핵심 개념 5: ConfigMap & Secret

Kubernetes는 애플리케이션 설정과 보안 정보를 컨테이너와 분리하여 관리한다.

### ConfigMap (컨피그맵)

- 애플리케이션 연동 및 접근 제어를 위한 **설정 내역**을 관리
- 컨테이너 이미지의 변경 없이 설정값을 업데이트할 수 있음
- 설정값을 외부로 노출하지 않고 사용 가능

### Secret (시크릿)

- **보안 키(패스워드, API 키, 인증서 등)** 를 안전하게 관리
- 컨테이너 이미지에 보안 정보를 포함하지 않아도 됨
- ConfigMap과 마찬가지로 이미지 변경 없이 업데이트 가능

> 📌 **핵심 포인트**  
> ConfigMap/Secret을 사용하면 **이미지 재빌드 없이** 설정/보안 정보를 변경할 수 있다.

---

## 🔑 핵심 개념 6: Service 타입 3가지 (⭐⭐ 최빈출!)

Pod의 IP는 계속 변하기 때문에, **안정적인 엔드포인트**를 제공하는 것이 Service.  
컨테이너에 IP 주소를 자동으로 할당하고, 클러스터 내 트래픽을 로드밸런싱할 수 있는 컨테이너 세트에 단일 DNS 이름을 할당한다.

```
[Service의 역할]
클라이언트 → Service (고정 IP/DNS) → [Pod1][Pod2][Pod3]
                                      (로드밸런싱)
```

### Service Type 1: ClusterIP (기본값)

```
인터넷 ────[차단]────  Service(ClusterIP)  →  Pod들
                         ↑
                  클러스터 내부에서만 접근 가능
```

- 클러스터 **내부**에서만 통신 가능
- 외부 인터넷에서 접근 불가
- 주로 내부 서비스 간 통신에 사용

### Service Type 2: NodePort

```
인터넷 ──→ Worker Node IP:30080 ──→ Service ──→ Pod들
               ↑
        노드의 30000~32767 포트를 외부에 개방
```

- 각 Worker Node의 특정 포트(30000~32767)를 열어 외부 접근 허용
- 노드 IP:포트번호로 외부에서 접근
- ClusterIP를 포함하는 개념

### Service Type 3: LoadBalancer

```
인터넷 ──→ NCP Load Balancer (공인 IP) ──→ Service ──→ Pod들
               ↑
        클라우드 제공자의 LB를 자동 연동
```

- **클라우드 제공자(NCP 등)의 외부 로드밸런서를 자동으로 연동**
- 공인 IP를 자동으로 할당
- NodePort를 포함하는 상위 개념

```
[Service Type 포함 관계]
LoadBalancer > NodePort > ClusterIP
```

---

## 🔑 핵심 개념 7: Ingress (인그레스)

**Ingress** = HTTP/HTTPS 요청을 URL 경로 또는 도메인에 따라 알맞은 Service로 라우팅하는 **L7 라우터**.

```
[Ingress 라우팅 예시]

https://bakery.com/          → Service A (웹 프론트엔드)
https://bakery.com/api/      → Service B (API 서버)
https://bakery.com/admin/    → Service C (관리자 페이지)
       ↑
 하나의 공인 IP로 여러 서비스 처리!
```

### Ingress 동작 조건

> ⚠️ **시험 포인트**  
> Ingress가 외부 트래픽을 받으려면 뒤에 연결된 Service 타입이 **반드시 NodePort 또는 LoadBalancer**여야 한다!  
> (ClusterIP는 외부 접근 불가이므로 Ingress 뒤에 쓸 수 없음)

### NCP에서의 Ingress 장점

```
[비용 절감 효과]
공인 IP 1개 (VIP)
   ├─ bakery.com → Service A
   ├─ api.bakery.com → Service B
   └─ admin.bakery.com → Service C

→ 도메인마다 공인 IP를 따로 쓰지 않아도 됨!
→ VIP 비용, 로드밸런서 비용 절감
```

---

## 🔑 핵심 개념 8: Kubernetes 주요 기능 8가지

교과서에서 명시한 쿠버네티스의 핵심 기능들이다.

| 기능 | 설명 |
|------|------|
| **Automatic Binpacking** | Worker Node의 가용성을 유지하면서 보유한 리소스를 충분히 활용하도록 스스로 스케줄링하여 컨테이너를 배치 |
| **Service Discovery & Load Balancing** | 컨테이너에 IP 주소를 자동으로 할당하고, 클러스터 내 트래픽을 로드밸런싱할 수 있는 컨테이너 세트에 단일 DNS 이름을 할당 |
| **Storage Orchestration** | 로컬 저장소 또는 NFS, iSCSI 등 공유 네트워크 스토리지를 컨테이너에 할당/마운트하여 사용 가능 |
| **Self Healing** | 실패한 컨테이너를 자동으로 다시 시작하고, 헬스체크에 응답 없는 컨테이너를 종료. 워커 노드 장애 시 다른 노드에 컨테이너를 다시 기동 |
| **Secret & Configuration Management** | 보안 키, 설정 내역을 컨테이너 이미지 변경 없이 업데이트할 수 있고, 외부로 노출하지 않고 사용 가능 |
| **Batch Execution** | 컨테이너 기반 서비스 관리뿐 아니라 배치 및 CI 작업 부하를 관리할 수 있으므로, 원하는 경우 실패한 컨테이너 대체 가능 |
| **Horizontal Scaling** | CPU 사용률 같은 metric을 기반으로 Pod의 ReplicaSet을 스케줄링하여 수평적 확장 가능 |
| **Automatic Rollbacks & Rollouts** | Deployment/컨테이너의 응용프로그램이나 구성에 대한 변경사항을 점진적으로 업데이트하고, 문제 발생 시 자동으로 롤백 |

---

## 🔑 핵심 개념 9: Kubernetes의 5가지 장점

1. **애플리케이션 배포 단순화** — 복잡한 배포 과정을 자동화
2. **하드웨어 활용도 극대화** — 클러스터 내 다양한 애플리케이션 구성요소를 가용 리소스에 최대한 맞춰 배치
3. **상태 확인 및 자가 치유** — 애플리케이션 구성요소와 노드를 모니터링하고, 노드 장애 발생 시 다른 노드로 자동 재조정
4. **오토스케일링** — 개별 애플리케이션 부하를 지속적으로 모니터링할 필요 없이 자동으로 각 컨테이너 수를 조정
5. **애플리케이션 개발 단순화** — 새 버전 출시 시 자동으로 테스트하고, 이상 발견 시 롤아웃 수행

---

## 전체 K8s 오브젝트 관계 정리

```
사용자/외부 트래픽
        ↓
    Ingress (L7 라우팅)
        ↓
    Service (LoadBalancer/NodePort/ClusterIP)
        ↓
    Deployment (배포 전략 + Horizontal Scaling)
        ↓
    ReplicaSet (Pod 개수 유지)
        ↓
    Pod (앱 실행 단위)
        ↓
    Container (실제 앱 프로세스)
        ↑
    ConfigMap / Secret (설정 및 보안 정보 주입)
```

---

## 📝 시험 대비 체크리스트

**Master Node:**
- [ ] **API Server**: 모든 요청의 통로 (사용자 및 컨트롤 플레인과 통신)
- [ ] **Controller-Manager**: 복제, 워커 노드 추적, 노드 장애 처리 등 클러스터 수준 기능
- [ ] **Scheduler**: 최적 Worker Node 선택 및 할당
- [ ] **etcd**: 클러스터 데이터 저장 (안정적인 분산 Key-Value DB)

**Worker Node:**
- [ ] **kubelet**: API Server와 통신하며 Pod 상태 체크/관리 에이전트
- [ ] **kube-proxy**: 애플리케이션 간 네트워크 트래픽 분산 및 연결
- [ ] **Container Runtime**: 컨테이너 실행 엔진

**오브젝트:**
- [ ] Pod: 최소 배포 단위, IP 변동
- [ ] ReplicaSet: Pod 개수 유지 (롤링 업데이트 없음)
- [ ] Deployment: 롤링 업데이트/롤백/수평 스케일링 담당
- [ ] DaemonSet: 모든 노드에 Pod 1개씩
- [ ] ConfigMap: 설정 내역 분리 관리 (이미지 변경 불필요)
- [ ] Secret: 보안 키 분리 관리 (외부 노출 없이 사용)
- [ ] ClusterIP: 내부 통신만
- [ ] NodePort: 노드 포트(30000~32767)로 외부 접근
- [ ] LoadBalancer: 클라우드 LB 자동 연동
- [ ] Ingress: L7 라우팅, 뒤에 NodePort/LoadBalancer 필요

**K8s 8대 기능:**
- [ ] Automatic Binpacking (자동 스케줄링/배치)
- [ ] Service Discovery & Load Balancing (IP 자동 할당, DNS)
- [ ] Storage Orchestration (로컬/NFS/iSCSI 마운트)
- [ ] Self Healing (자동 재시작, 헬스체크)
- [ ] Secret & Configuration Management (이미지 변경 없이 설정 업데이트)
- [ ] Batch Execution (배치/CI 작업 지원)
- [ ] Horizontal Scaling (CPU metric 기반 수평 확장)
- [ ] Automatic Rollbacks & Rollouts (점진적 배포 및 자동 롤백)

---

## 🧠 암기 핵심 문장

> "**API Server가 먼저 받고, Scheduler가 배치하고, etcd가 기억한다.**"  
> "**Deployment > ReplicaSet > Pod — 계층 구조를 기억!**"  
> "**ClusterIP = 내부, NodePort = 포트 열기, LoadBalancer = 클라우드 LB.**"  
> "**Ingress 뒤에는 반드시 NodePort 또는 LoadBalancer.**"  
> "**ConfigMap = 설정, Secret = 보안 키 — 둘 다 이미지 변경 없이 업데이트.**"  
> "**K8s 8대 기능: Binpacking, ServiceDiscovery, Storage, SelfHealing, SecretConfig, Batch, HPA, Rollout.**"

---

*다음 챕터: `12_NKS.md` → NCP 관리형 쿠버네티스*
