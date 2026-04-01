---
title: "Chapter 11: 주방 총관리자 — Kubernetes (K8s)"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, kubernetes, k8s, pod, deployment, service]
description: "NCP NCE 자격증 대비 학습 자료 - Kubernetes 핵심 컴포넌트와 오브젝트 구조 정리"
---
> **학습 목표**: K8s 컴포넌트(Master/Worker 구분), Pod/ReplicaSet/Deployment 계층, Service 타입 3가지, DaemonSet, Ingress를 완벽히 구분한다.

---

## 이야기: 컨테이너가 100개가 되면?

클라우드빵집이 폭발적으로 성장했다.
컨테이너가 처음엔 5개, 다음엔 50개, 어느새 100개.

> "이걸 다 수동으로 관리해? 어느 서버에 올리고, 하나 죽으면 다시 살리고..."

개발팀 리더가 말했다.

> "**쿠버네티스(Kubernetes)** 써. 컨테이너 오케스트레이션 도구야.
> 스케줄링, 자동 복구, 스케일링을 알아서 해줘."

---

## 🔑 핵심 개념 1: 컨테이너 오케스트레이션

컨테이너가 많아지면 발생하는 문제들:
- 어느 서버에 올릴지 결정 (스케줄링)
- 컨테이너가 죽으면 자동 재시작 (자가 복구)
- 트래픽 증가 시 컨테이너 수 늘리기 (스케일링)
- 트래픽 분산 (로드 밸런싱)

이런 문제를 **자동화**해주는 도구 = **컨테이너 오케스트레이션**  
대표 도구: **Kubernetes**, Docker Swarm, Apache MESOS

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

---

### Master Node 컴포넌트

| 컴포넌트 | 역할 |
|---------|------|
| **API Server** | 모든 요청을 받는 **핵심 통로** (사용자, 컴포넌트 모두 API Server를 통해 통신) |
| **Controller-Manager** | 클러스터 상태 관리 및 유지 (복제, 배포, 상태 확인) |
| **Scheduler** | Pod를 실행할 **최적의 Worker Node를 선택(배치)** |
| **etcd** | 클러스터의 모든 설정/상태 데이터를 저장하는 **분산 Key-Value DB** |

> 📌 **시험 단답형 패턴**  
> "사용자 요청을 제일 먼저 받는 곳?" → **API Server**  
> "어떤 노드에 배포할지 결정하는 곳?" → **Scheduler**  
> "클러스터 상태 데이터를 저장하는 곳?" → **etcd**

---

### Worker Node 컴포넌트

| 컴포넌트 | 역할 |
|---------|------|
| **kubelet** | Master의 명령을 받아 **Pod 상태를 체크하고 관리**하는 에이전트 |
| **kube-proxy** | Pod로 연결되는 **네트워크 및 로드밸런싱 규칙** 관리 |
| **Container Runtime** | 실제로 컨테이너를 실행하는 소프트웨어 (예: Docker, containerd) |

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
- Pod 하나가 죽으면 즉시 새 Pod를 생성
- **한계**: 롤링 업데이트(무중단 배포) 기능 없음

### Deployment (디플로이먼트)

- ReplicaSet을 감싸는 상위 개념
- **롤링 업데이트(Rolling Update)** 와 **롤백(Rollback)** 담당
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

## 🔑 핵심 개념 5: Service 타입 3가지 (⭐⭐ 최빈출!)

Pod의 IP는 계속 변하기 때문에, **안정적인 엔드포인트**를 제공하는 것이 Service.

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

## 🔑 핵심 개념 6: Ingress (인그레스)

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

## 전체 K8s 오브젝트 관계 정리

```
사용자/외부 트래픽
        ↓
    Ingress (L7 라우팅)
        ↓
    Service (LoadBalancer/NodePort/ClusterIP)
        ↓
    Deployment (배포 전략)
        ↓
    ReplicaSet (Pod 개수 유지)
        ↓
    Pod (앱 실행 단위)
        ↓
    Container (실제 앱 프로세스)
```

---

## 📝 시험 대비 체크리스트

**Master Node:**
- [ ] **API Server**: 모든 요청의 통로
- [ ] **Controller-Manager**: 클러스터 상태 관리
- [ ] **Scheduler**: 최적 Worker Node 선택
- [ ] **etcd**: 클러스터 데이터 저장 (분산 Key-Value DB)

**Worker Node:**
- [ ] **kubelet**: Pod 상태 체크/관리 에이전트
- [ ] **kube-proxy**: 네트워크/로드밸런싱 규칙
- [ ] **Container Runtime**: 컨테이너 실행 엔진

**오브젝트:**
- [ ] Pod: 최소 배포 단위, IP 변동
- [ ] ReplicaSet: Pod 개수 유지 (롤링 업데이트 없음)
- [ ] Deployment: 롤링 업데이트/롤백 담당
- [ ] DaemonSet: 모든 노드에 Pod 1개씩
- [ ] ClusterIP: 내부 통신만
- [ ] NodePort: 노드 포트(30000~32767)로 외부 접근
- [ ] LoadBalancer: 클라우드 LB 자동 연동
- [ ] Ingress: L7 라우팅, 뒤에 NodePort/LoadBalancer 필요

---

## 🧠 암기 핵심 문장

> "**API Server가 먼저 받고, Scheduler가 배치하고, etcd가 기억한다.**"  
> "**Deployment > ReplicaSet > Pod — 계층 구조를 기억!**"  
> "**ClusterIP = 내부, NodePort = 포트 열기, LoadBalancer = 클라우드 LB.**"  
> "**Ingress 뒤에는 반드시 NodePort 또는 LoadBalancer.**"

---

*다음 챕터: `12_NKS.md` → NCP 관리형 쿠버네티스*
