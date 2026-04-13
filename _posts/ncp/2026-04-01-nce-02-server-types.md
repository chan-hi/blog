---
title: "Chapter 2: 주방 규모를 정하다 — 서버 타입"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, server, micro, standard, high-memory, high-cpu, cpu-intensive]
description: "NCP NCE 자격증 대비 학습 자료 - 서버 타입별 CPU:Memory 비율과 용도 완벽 정리"
---

> **학습 목표**: 각 서버 타입별 CPU:Memory 비율과 용도를 완벽히 매칭한다.

---

## 이야기: 메뉴판처럼 늘어선 서버 타입들

G3 서버를 만들기로 한 민준. 이번엔 서버 **타입**을 골라야 했다.

```
[ 서버 타입 선택 ]
□ Micro
□ Compact
□ Standard
□ High Memory
□ High CPU
□ CPU-Intensive
□ GPU
□ ...
```

> "다 뭐가 다른 거야? 그냥 제일 좋은 거 쓰면 안 돼?"

지수가 웃으며 말했다.

> "용도에 맞게 써야 돈 낭비가 없어. 각 타입마다 **CPU와 Memory의 비율**이 달라.
> 어떤 작업을 하느냐에 따라 뭐가 더 많이 필요한지 달라지거든."

---

## 서버 세대: G1 / G2 / G3

NCP 서버는 세대별로 구분된다. 세대가 다르면 하이퍼바이저도 달라진다.

| 세대 | 하이퍼바이저 | 서버 타입 | 스토리지 |
|------|------------|----------|---------|
| **G1** | XEN | Micro, Standard, High Memory, High CPU | HDD, SSD |
| **G2** | XEN | Micro, Standard, High Memory, High CPU | HDD, SSD |
| **G3** | KVM | Micro, Standard, High Memory, High CPU | FB1, CB1, FB2, CB2 |

> ⚠️ **중요**: 하이퍼바이저가 다른 리소스(서비스) 간 호환은 불가능하다.

G3(KVM)에서는 **AMD 기반 서버**도 제공된다 (High CPU, High Memory, Standard).  
G3는 G1/G2(XEN)에 비해 CPU 성능이 비약적으로 발전했다.

---

## 서버 핵심 스펙

### CPU
- **vCPU** 단위로 할당
- CPU의 Core가 vCPU에 매핑되는 구조
- **OverCommit 허용** (물리 코어보다 더 많은 vCPU 할당 가능)
- 스펙 변경 가능

### Memory
- **GB** 단위로 할당
- **OverCommit 허용 안 함** (메모리는 물리 용량 이상 초과 할당 불가)
- 스펙 변경 가능

### Block Storage (OS 디스크)
- XEN 기반: OS 영역은 **용량 확장/축소 불가** (Linux 50GB / Windows 100GB 고정)
- KVM 기반: **서버 정지 후** OS 영역 스토리지 확장 가능
  - Linux: 최소 10GB ~ 최대 2TB
  - Windows: 최소 30GB ~ 최대 2TB
- 추가 스토리지: 10GB ~ 최대 **16TB**까지 생성 가능
- 추가 스토리지는 최대 **20개**까지 생성 가능 (기본 스토리지 포함 서버당 21개)

> ⚠️ **시험 포인트**  
> Windows 서버는 기본 디스크로 **100GB** 서비스를 활용할 수 있다.

### Network
- Physical: **10Gbps**
- Logical: **1Gbps**
- 실제 제공: **500Mbps ~ 1Gbps** (RX+TX 합산)

---

## 🔑 핵심 개념: 서버 타입과 CPU:Memory 비율

NCP 서버 타입은 **vCPU : Memory(GB)** 비율로 구분된다.

| 타입 | **vCPU : Memory** | 스펙 예시 | 주요 용도 | 특이사항 |
|------|------------------|---------|----------|---------|
| **Micro** | **1 : 1** | vCPU 1개, 메모리 1GB | 테스트, 개인 프로젝트 | **1년간 무료** 제공 (이후 과금) |
| **Standard** | **1 : 4** | s2: vCPU 2개, 메모리 8GB | 일반 웹서버, DB | 가장 기본적인 범용 타입 |
| **High Memory** | **1 : 8** | m4: vCPU 4개, 메모리 32GB | 고성능 DB, 인메모리 캐시 | 메모리가 CPU 대비 8배 |
| **High CPU** | **1 : 2** | c16: vCPU 16개, 메모리 32GB | 과학적 모델링, 컴퓨팅 집약 | CPU 비율이 높음 |
| **CPU-Intensive** | **1 : 2** | c16: vCPU 16개, 메모리 32GB | 딥러닝, 고성능 웹서버 | 많은 연산 처리 특화 |
| **GPU** | — | T4/V100/A100 탑재 | 딥러닝 학습/추론 | PassThrough 방식 제공 |

> ⚠️ **시험 포인트**  
> **High CPU**와 **CPU-Intensive** 둘 다 1:2 비율이지만 용도가 다르다.  
> High CPU → 과학적 모델링 / CPU-Intensive → 딥러닝, 고성능 웹서버

---

## 서버 타입별 기능 지원 현황

페이지 52의 서버 타입별 특성표를 정리하면:

| 서버 타입 | 추가 스토리지 | 스토리지 크기 변경 | SSD 이용 | 오토스케일링 | 네트워크 인터페이스 |
|----------|------------|----------------|---------|-----------|----------------|
| Micro (G3) | O | O | O | X | X |
| Standard (G3, G2) | O | O | O | O | O |
| High Memory (G3, G2) | O | O | O | O | O |
| High CPU (G3, G2) | O | O | O | O | O |
| CPU-Intensive (G2) | O | O | O | O | O |
| GPU | O | O | O | X | X |
| Bare Metal Server | X | X | O | X | — |

> ⚠️ **시험 포인트**  
> **Micro**는 오토스케일링과 네트워크 인터페이스 추가가 **불가**하다.  
> **Bare Metal Server**는 추가 스토리지와 스토리지 크기 변경이 **불가**하다.

---

## GPU 서버

딥러닝과 병렬 연산이 필요할 때 선택한다.

- 병렬 처리에 최적화된 고성능 컴퓨팅 파워 제공
- **NVIDIA GRID 기술이 아닌 PassThrough** 방식으로 제공 (성능 저하 없음)
- 탑재 GPU 종류: **NVIDIA T4, V100, A100**

| 서버 형태 | 탑재 가능 GPU |
|---------|-------------|
| VM (가상 서버) | T4 (최대 2장), V100 (최대 4장) |
| Bare Metal Server | A100 (최대 8장) + 로컬 NVMe 디스크 기본 지원 |

**지원 OS**:
- CentOS 7.3 (T4 미지원) & 7.8
- Ubuntu 16.04 & 18.04 (P40 제외) & 20.04 (A100만 제공)
- Windows 2016 & 2019

---

## 타입별 선택 가이드

```
어떤 작업을 하나요?

무료로 써보고 싶다
  └→ Micro (1:1) — 1년 무료, 이후 과금 / 단, 오토스케일링 불가

일반적인 웹서버, 중소형 DB
  └→ Standard (1:4) — 가장 많이 씀

수백만 건 쿼리를 빠르게 처리하는 DB
Redis, Elasticsearch 같은 인메모리
  └→ High Memory (1:8)

기상 예측, 유전자 분석, 물리 시뮬레이션
  └→ High CPU (1:2)

딥러닝 전처리, 트래픽 폭발적인 웹서버
  └→ CPU-Intensive (1:2)

딥러닝 학습/추론, 병렬 GPU 연산
  └→ GPU 서버 (T4/V100/A100)
```

---

## 이야기: 민준의 선택

민준은 고민 끝에 결정했다.

```
[클라우드빵집 서버 구성]

웹 애플리케이션 서버: Standard (1:4)
  → 일반적인 HTTP 요청 처리
  → 스펙 예시: s2 (vCPU 2개, 메모리 8GB)

주문 데이터베이스 서버: High Memory (1:8)
  → 수많은 주문 쿼리, JOIN 연산 처리
  → 스펙 예시: m4 (vCPU 4개, 메모리 32GB)

AI 추천 시스템 (나중에): CPU-Intensive (1:2)
  → 딥러닝 추론 처리
  → 스펙 예시: c16 (vCPU 16개, 메모리 32GB)
```

> "처음엔 Micro로 테스트하고, 실서비스는 Standard로 가야겠다.  
> DB는 High Memory가 맞는 것 같고."

---

## 🧠 암기법

```
Micro  → 1:1  (혼자 일하는 작은 가게 — 1년은 공짜)
Standard → 1:4 (점원 1명이 테이블 4개 담당 — 기본 세팅)
High Memory → 1:8 (기억력 좋은 바리스타 — 모든 주문 외움)
High CPU / CPU-Intensive → 1:2 (빠른 손을 가진 요리사)
GPU → 병렬 처리 전문 주방 (모든 버너 동시 가동)
```

**스펙 변경 규칙 암기**:
- CPU → OverCommit **O** (초과 할당 가능)
- Memory → OverCommit **X** (초과 할당 불가)

---

## 📝 시험 대비 체크리스트

- [ ] Micro: **1:1** 비율, **1년 무료** (이후 과금), 오토스케일링 **불가**
- [ ] Standard: **1:4** 비율, 일반 웹/DB, 스펙 예시: s2(vCPU 2, 메모리 8GB)
- [ ] High Memory: **1:8** 비율, 고성능 DB/인메모리, 스펙 예시: m4(vCPU 4, 메모리 32GB)
- [ ] High CPU: **1:2** 비율, 과학적 모델링/컴퓨팅 집약, 스펙 예시: c16(vCPU 16, 메모리 32GB)
- [ ] CPU-Intensive: **1:2** 비율, 딥러닝/고성능 웹서버, 스펙 예시: c16(vCPU 16, 메모리 32GB)
- [ ] High CPU와 CPU-Intensive는 비율은 같지만 용도가 다름
- [ ] GPU 서버: PassThrough 방식, T4/V100/A100, VM은 T4(최대 2장)/V100(최대 4장)
- [ ] G1/G2 = XEN 기반, G3 = KVM 기반
- [ ] CPU OverCommit 허용, **Memory OverCommit 허용 안 함**
- [ ] XEN: OS 디스크 크기 고정 (Linux 50GB, Windows 100GB)
- [ ] KVM: OS 디스크 크기 변경 가능 (서버 정지 후)
- [ ] 추가 스토리지: 최대 16TB, 최대 20개 (기본 포함 총 21개)
- [ ] Windows 서버: 기본 디스크 **100GB** 서비스 활용 가능
- [ ] 네트워크: Physical 10Gbps, Logical 1Gbps

---

## 🧠 암기 핵심 문장

> "**메모리가 중요하면 High Memory(1:8), CPU가 중요하면 1:2, 기본은 Standard(1:4).**"

> "**CPU는 OverCommit 가능, Memory는 OverCommit 불가 — 메모리는 철저하게 관리된다.**"

---

*다음 챕터: `03_특수서버.md` → 베어메탈과 GPU — 더 강한 주방*
