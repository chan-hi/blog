---
title: "[NCE] 04. 컨테이너, Kubernetes, Auto Scaling"
date: 2026-05-14 09:30:00 +0900
categories: [NCP, NCE]
tags: [NCP, NCE, Docker, Kubernetes, NKS, Auto Scaling, 컨테이너, Pod, Deployment]
description: "VM vs Container 비교, Docker 기반 기술, Kubernetes 아키텍처, NKS 구성, Auto Scaling 정책 정리"
---

## 핵심 요약

- Container는 Guest OS 없이 Host OS 커널 공유 → VM보다 오버헤드 적음
- Docker 기반 기술: Namespace(격리), cgroups(리소스 제한), Union Filesystem(Layer)
- K8s Control Plane: API Server / etcd / Scheduler / Controller Manager
- ReplicaSet은 롤링 업데이트 미지원, Deployment는 롤백 지원
- DaemonSet: 모든 노드에 Pod 1개씩 보장
- NKS: Master Hidden/HA 3대, Worker 최대 250대
- Scale-out: 서버 대수 늘림, Launch Configuration: 생성될 서버 템플릿

## VM vs Container 비교

| 항목 | VM (가상머신) | Container |
|---|---|---|
| 격리 방식 | Hypervisor + Guest OS 전체 가상화 | Host OS 커널 공유, 프로세스 수준 격리 |
| OS | 각 VM마다 독립된 Guest OS | Host OS 커널 공유 (Guest OS 없음) |
| 오버헤드 | 크다 | **작다** |
| 기동 속도 | 느림 (OS 부팅 필요) | **빠름** (프로세스 시작) |

## Docker 핵심 기반 기술

| 기술 | 역할 |
|---|---|
| **Linux Namespace** | 프로세스가 파일시스템, 네트워크, 유저 등에 대해 독립 뷰 제공 (격리) |
| **Linux cgroups** | CPU, 메모리, I/O, 네트워크 등 리소스 양 제한 |
| **Union Filesystem** | 여러 Layer를 하나의 파일시스템으로 인식 |

- **Layered Image**: 변경분만 기록, 최상단에 R/W layer 존재
- 공통 베이스 이미지 Layer를 컨테이너들이 **공유** → 디스크 효율화
- Docker 공식 레지스트리: **Docker Hub**

### Dockerfile 주요 명령어

| 명령어 | 설명 |
|---|---|
| `FROM` | 베이스 이미지 지정 |
| `RUN` | 컨테이너에서 명령 실행 |
| `CMD` | 기본 시작 명령 지정 |
| `EXPOSE` | 컨테이너가 공개하는 포트 선언 |
| `COPY` / `ADD` | 파일 복사 (ADD는 URL, tar 지원) |
| `ENTRYPOINT` | 데몬 실행 |
| `ENV` | 환경 변수 설정 |

## NCP Container Registry

- Docker 이미지를 저장, 관리, 배포하는 서비스
- **Object Storage에 저장** → 안정성 및 확장성 제공
- **CVE 기반 취약점 분석** 제공
- Public/Private Endpoint 제공

## Kubernetes 아키텍처

### Control Plane 컴포넌트

| 컴포넌트 | 역할 |
|---|---|
| **API Server** | 모든 요청을 처리하는 인터페이스 |
| **etcd** | 클러스터 데이터를 담는 분산 저장소 |
| **Scheduler** | 적절한 Worker Node를 선택 |
| **Controller Manager** | 복제/배포/상태 등 다양한 컨트롤러 관리 |

### Worker Node 컴포넌트

| 컴포넌트 | 역할 |
|---|---|
| **kubelet** | Node에 할당된 Pod 상태 체크 및 관리 |
| **kube-proxy** | Pod로 연결되는 네트워크 관리 |
| **Container Runtime** | 배포된 컨테이너 실행 |

## Kubernetes 핵심 개념

### Pod
- **가장 작고 간단한 배포 단위**
- 하나 이상의 컨테이너로 구성
- 각 Pod는 자체 IP, 호스트이름, 프로세스 보유

### ReplicaSet
- 대상 Pod의 복제본을 지정한 수만큼 유지
- **롤링 업데이트 미지원**

### Deployment
- ReplicaSet을 그룹화하고 관리
- 배포 이력 관리 → **이전 버전으로 롤백 가능**
- 전략: Rolling Update, Blue/Green, Canary

### DaemonSet
- **클러스터 내 모든 노드에서 Pod 1개씩** 실행 보장
- 노드 추가 시 Pod 자동 추가, 노드 삭제 시 Pod 삭제

### Controller 계층

```
Deployment (배포 이력 관리 + 롤백)
    └── ReplicaSet (Pod 복제본 집합 관리)
            └── Pod (컨테이너 실행 단위)

DaemonSet (모든 노드에 Pod 1개씩)
```

### Service Type

| 타입 | 설명 |
|---|---|
| **ClusterIP** | 기본. 클러스터 **내부에서만** 사용 가능 |
| **NodePort** | **30000~32768 포트** 이용, Pod로 트래픽 포트 포워딩 |
| **LoadBalancer** | CSP의 External LB를 통해 외부 트래픽 수신 |

### Ingress
- 외부 트래픽을 클러스터 내부 Service로 라우팅하는 규칙 집합
- Host 기반 및 URL Path 기반 라우팅
- Service 타입은 **NodePort 혹은 LoadBalancer** 지정 필요

## NKS (Ncloud Kubernetes Service)

- **완전 관리형 Kubernetes Cluster**
- Control Plane 관리 불필요, 사용자는 Worker Node만 관리

| 항목 | Legacy Kubernetes | NKS |
|---|---|---|
| 마스터 노드 | 직접 구성/관리 | **Hidden 구성, 관리 불필요** |
| 고가용성 | 직접 확보 | **HA 제공 (Master 3대)** |
| 워커 노드 | 직접 준비 | **수분 안에 구성** |
| 스토리지 | 별도 구성 | **NAS, Block Storage 사용 가능** |

- **Worker Node 최대 250대**
- Auto Install Addon: CSI for Block Storage, WeaveScope, Ingress Nginx, **Cluster AutoScaler**

## Auto Scaling

### Scale-up/down vs Scale-out/in

| 방식 | 설명 |
|---|---|
| **Scale-up** | 서버 사양을 높임 (수직 확장) |
| **Scale-down** | 서버 사양을 낮춤 (수직 축소) |
| **Scale-out** | **서버 대수를 늘림** (수평 확장, 무중단 가능) |
| **Scale-in** | 서버 대수를 줄임 (수평 축소) |

> Scale-in/out 작동을 위해 **N-tier Architecture** 사전 고려 필요

### Auto Scaling Group 구성 요소

- **Auto Scaling Group**: Auto Scaling의 기본 단위 (Group명 + Launch Configuration명 + 최소/최대/기본 용량)
- **Launch Configuration**: 생성될 서버의 템플릿 (서버 이미지, 타입, 인증키, ACG, 스토리지 등)

### 스케일링 트리거

1. **Metric 기반**: CPU/메모리 사용률 등
2. **일정 기반**: 특정 시간대에 맞춰 자동 조정
3. **수동**: 관리자 직접 조정

## 시험 암기 포인트

- **VM vs Container**: Container는 Guest OS 없음, Host OS 커널 공유
- **Docker 기반**: Namespace (격리) + cgroups (리소스 제한) + Union FS (Layer)
- **Control Plane 4종**: API Server / etcd / Scheduler / Controller Manager
- **Worker Node 3종**: kubelet / kube-proxy / Container Runtime
- **ReplicaSet**: 롤링 업데이트 **미지원** / **Deployment**: 롤백 지원
- **DaemonSet**: 모든 노드에 Pod 1개
- **Service NodePort**: 30000~32768 포트
- **NKS**: Master Hidden + HA 3대, Worker 최대 250대
- **Scale-out**: 서버 대수 늘림 / **Launch Configuration**: 서버 템플릿

