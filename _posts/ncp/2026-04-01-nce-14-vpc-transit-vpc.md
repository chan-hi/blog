---
title: "Chapter 14: 나만의 클라우드 구역 — VPC & Transit VPC"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, vpc, transit-vpc, peering, routing, network]
description: "NCP NCE 자격증 대비 학습 자료 - VPC 구조와 VPC Peering, Transit VPC 차이 정리"
---
> **학습 목표**: VPC 기본 개념, VPC Peering vs Transit VPC 차이, Transit VPC의 제약을 완벽히 이해한다.

---

## 이야기: 클라우드빵집 전용 사무실을 만들다

클라우드빵집이 커지면서, 다른 회사들과 네트워크를 공유하는 것이 불안해졌다.

> "우리 서버들끼리만 통신하는 전용 네트워크가 있으면 좋겠어.
> 외부에선 접근 못 하도록."

재현이 설명했다.

> "그게 VPC야. Virtual Private Cloud.
> 클라우드 안에 '나만의 사설 네트워크'를 만드는 거야.
> 다른 고객과 완전히 격리된 공간이지."

---

## 🔑 핵심 개념 1: VPC (Virtual Private Cloud)

**VPC** = 클라우드 환경에서 논리적으로 격리된 **나만의 사설 네트워크**.

```
[VPC 구조]

┌─────────────────────────────────────────┐
│ VPC (나만의 클라우드 구역)               │
│  CIDR: 10.0.0.0/16                      │
│                                          │
│  ┌────────────────┐  ┌────────────────┐  │
│  │ Public Subnet  │  │ Private Subnet │  │
│  │ 10.0.1.0/24   │  │ 10.0.2.0/24   │  │
│  │ (웹 서버)     │  │ (DB 서버)     │  │
│  └────────────────┘  └────────────────┘  │
│                                          │
└─────────────────────────────────────────┘
```

### VPC 핵심 구성 요소

| 구성 요소 | 역할 |
|---------|------|
| **Subnet** | VPC를 더 작은 IP 대역으로 나눈 단위 |
| **Route Table** | 네트워크 트래픽 경로를 결정하는 라우팅 규칙 |
| **Internet Gateway** | VPC ↔ 인터넷 연결 게이트웨이 |
| **NAT Gateway** | Private Subnet → 인터넷 단방향 연결 |
| **NACL** | Subnet 단위 방화벽 (Stateless) |
| **ACG** | 서버 NIC 단위 방화벽 (Stateful) |

---

## 🔑 핵심 개념 2: VPC Peering vs Transit VPC (⭐ 시험 단골!)

클라우드빵집이 자회사 "빵집배달"의 VPC와 연결해야 할 상황이 생겼다.

### VPC Peering (1:1 연결)

```
VPC A ─────── VPC B
        (1:1)

VPC A ─────── VPC B
     \─────── VPC C
              (추가 연결 시 각각 Peering 설정)
```

- VPC 간 **1:1 연결** 구조
- 여러 VPC 연결 시 연결 수가 기하급수적 증가
- 관리 복잡도 높음

### Transit VPC (허브 역할, 1:N 연결)

```
        ┌─────────────────┐
        │   Transit VPC   │  ← 중앙 허브
        │  (Hub 역할)      │
        └────┬────┬────┬──┘
             │    │    │
           VPC A VPC B VPC C
           (1:N 연결 — 모두 Transit을 통해 연결)
```

### Transit VPC 특징 (⭐ 필수 암기!)

| 항목 | 내용 |
|------|------|
| **역할** | VPC ↔ VPC, VPC ↔ 인터넷, VPC ↔ 온프레미스 연결 허브 |
| **구조** | **1:N** 연결 (VPC Peering은 1:1) |
| **생성 개수** | 메인 계정당 **최대 1개** |
| **연계** | Endpoint Route, Transit VPC Connect와 함께 라우팅 흐름 제어 |

> ⚠️ **시험 포인트**  
> Transit VPC: 메인 계정당 **최대 1개**만 생성 가능!  
> VPC Peering: 1:1 / Transit VPC: **1:N**

---

## 🔑 핵심 개념 3: 서브넷 설계 전략

```
[클라우드빵집 VPC 설계]

VPC: 10.0.0.0/16 (65,534개 IP)
  │
  ├─ Public Subnet: 10.0.1.0/24 (254개 IP)
  │   └─ 웹 서버, 로드밸런서 (인터넷 접근 가능)
  │
  ├─ Private Subnet: 10.0.2.0/24 (254개 IP)
  │   └─ DB 서버, 앱 서버 (외부 접근 불가)
  │
  └─ Private Subnet: 10.0.3.0/24 (254개 IP)
      └─ 관리 서버, 모니터링 (내부 전용)
```

**Public vs Private Subnet:**
- **Public Subnet**: Internet Gateway와 연결 → 외부 인터넷 접근 가능
- **Private Subnet**: Internet Gateway 없음 → 외부 접근 불가, NAT Gateway로 아웃바운드만 허용

---

## 🔑 핵심 개념 4: NAT Gateway

Private Subnet의 서버가 인터넷(외부)과 통신할 때 필요.

```
[NAT Gateway 흐름]

Private Subnet (DB 서버: 10.0.2.10)
    ↓ 패키지 업데이트 요청
NAT Gateway (Public Subnet에 위치)
    ↓ 공인 IP로 변환
인터넷 (패키지 서버)
    ↓ 응답
NAT Gateway → Private Subnet

→ Private 서버는 외부로 요청 가능
→ 외부에서 Private 서버로 직접 접근은 불가
```

---

## 📝 시험 대비 체크리스트

- [ ] VPC: 논리적으로 격리된 사설 클라우드 네트워크
- [ ] VPC Peering: **1:1** 연결
- [ ] Transit VPC: **1:N** 연결, 허브 역할
- [ ] Transit VPC: 메인 계정당 **최대 1개**
- [ ] Public Subnet: Internet Gateway 연결, 외부 접근 가능
- [ ] Private Subnet: 외부 접근 불가, NAT Gateway로 아웃바운드만
- [ ] NACL: Subnet 방화벽 / ACG: NIC 방화벽

---

## 🧠 암기 핵심 문장

> "**Transit VPC는 허브: 1:N 연결, 계정당 1개.**"  
> "**Peering은 1:1, Transit은 1:N.**"

---

*다음 챕터: `15_로드밸런서.md` → 트래픽을 균등하게 나눠라*
