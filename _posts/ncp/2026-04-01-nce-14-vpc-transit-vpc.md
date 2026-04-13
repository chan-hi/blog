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

**VPC** = 클라우드 환경에서 **논리적으로 격리된 고객 전용 네트워크 공간**.

### VPC 기본 스펙 (⭐ 시험 단골!)

| 항목 | 내용 |
|------|------|
| **적용 리전** | 한국, 일본, 싱가포르 |
| **계정당 최대 생성 수** | **최대 3개** |
| **IP 주소 범위** | RFC 1918 대역 (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) |
| **Netmask 범위** | 최소 /28 ~ 최대 /16 |

> ⚠️ **시험 포인트**  
> VPC는 계정당 **최대 3개**까지만 생성 가능하며, 한국·일본·싱가포르 리전에 적용된다.

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

### VPC의 주요 특징

**전용 네트워크 사용**: 다른 네트워크와 상호 간섭 없이 논리적으로 완전히 분리된 네트워크를 사용할 수 있다.

**다양한 네트워크 토폴로지**: VPC 내부에 Public Subnet 또는 Private Subnet을 생성하여 고객 맞춤형 네트워크 환경을 조성할 수 있다. Subnet을 생성한 후 서버, 데이터베이스와 같은 리소스를 배치한다.

**강력한 보안**: ACG(Access Control Group)와 Network ACL(Network Access Control List)을 통해 네트워크 접근을 제어한다. ACG는 서버 단계의 접근을 제어하며, Network ACL은 Subnet 단계의 접근을 제어한다.

**VPC 간 내부 통신**: VPC Peering을 사용해 다른 VPC와 통신할 수 있다. 공인 IP 없이 내부 네트워크로 통신하기 때문에 비용 효율성이 높다.

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

## 🔑 핵심 개념 2: Subnet 상세

### Subnet 속성

- VPC 주소 범위 내에서 CIDR 형태로 주소 범위 지정
- Zone을 지정할 수 있으며, 동일한 Zone 내에 여러 Subnet 생성 가능
- **VPC당 최대 200개의 Subnet** 생성 가능
- 인터넷과 연결되는 **Public Subnet**과 폐쇄적인 **Private Subnet**으로 구분
- Public Subnet 내에 있는 서버만 공인 IP 부여 가능

### Subnet 종류

| 종류 | 설명 |
|------|------|
| **Public Subnet** | 외부 인터넷 통신이 가능한 서브넷. Internet Gateway Y 선택 시 생성되며, 이곳에 생성한 서버는 공인 IP를 부여받아 인터넷 통신 가능 |
| **Private Subnet** | 외부 인터넷 통신이 불가능한 서브넷. Internet Gateway N 선택 시 생성. 인터넷에 접속하려면 NAT Gateway 별도 생성 필요 (Outbound 통신만 가능) |
| **Load Balancer Subnet** | 로드밸런서 전용 서브넷 |
| **Baremetal 전용 Subnet** | 베어메탈 서버 전용 서브넷 |
| **NAT Gateway 전용 Subnet** | NAT Gateway 전용 서브넷 |

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

### Route Table 설정 예시

VPC Peering 연결 시 Route Table에 다음과 같이 설정한다.

| 목적지 | Target 유형 | Target 이름 |
|--------|------------|------------|
| VPC B의 IP 범위 (예: 172.31.0.0/16) | VPC PEERING | 설정한 VPC Peering 이름 |
| 0.0.0.0/0 | IGW | INTERNET GATEWAY |
| 10.0.0.0/16 | LOCAL | LOCAL |

---

## 🔑 핵심 개념 3: NACL (Network ACL)

- **Subnet 단위**의 방화벽 서비스
- Inbound/Outbound 트래픽의 허용/차단 규칙 모두 설정, 기본적으로 허용 규칙 적용
- **Stateless 방식** — 트래픽 상태를 저장하지 않아 Inbound 규칙과 Outbound 규칙을 별도 설정 필요
- 트래픽 허용 여부 결정 시 규칙을 **우선순위**로 처리
- 대상 Subnet 내 모든 서버에 적용 (ACG를 지정하는 사용자에게 의존할 필요 없음)

### Deny-Allow Group

- 다수의 IP 그룹으로, Network ACL의 Inbound/Outbound 규칙 설정 시 접근 소스 또는 목적지 대상으로 이용
- **VPC 1개당 최대 4개**의 Deny-Allow Group 생성 가능

---

## 🔑 핵심 개념 4: VPC Peering vs Transit VPC (⭐ 시험 단골!)

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
- **내 VPC 간 연결**뿐만 아니라 **다른 계정과의 VPC 연결도 가능**
- 타 계정 VPC 연결 시 로그인 ID, VPC ID, VPC 명 입력 필요
- 여러 VPC 연결 시 연결 수가 기하급수적으로 증가
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

- VPC와 VPC, VPC와 인터넷, VPC와 온프레미스 간 네트워크 상호 연결을 위한 **중앙 허브** 역할을 하는 VPC
- **Endpoint Route**, **Transit VPC Connect**와 연계하여 라우팅을 통해 연결(흐름) 제어 가능

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

## 🔑 핵심 개념 5: NAT Gateway

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

## 🔑 핵심 개념 6: Packet Mirroring

**Packet Mirroring**은 VPC 네트워크에서 지정된 서버(Network Interface)의 패킷을 캡처하고, 원하는 서버로 전송할 수 있는 기능이다.

- 패킷 미러링을 통해 **네트워크 트래픽을 기반으로 한 보안 강화** 가능
- 구성 요소:

| 구성 요소 | 역할 |
|----------|------|
| **Mirror Source** | 패킷 미러링 대상 (Network Interface) |
| **Mirror Destination** | 미러링된 패킷의 목적지 (Network Interface) |
| **Mirror Filter** | 패킷 미러링 필터 정책 생성 및 관리 (Inbound/Outbound Rules) |
| **Mirror Tunnel** | Mirror Filter를 기반으로 Tunnel을 통해 전송 |

```
[Packet Mirroring 흐름]

Mirror Source                  Mirror Destination
(Network Interface)            (Network Interface)
       │                              ↑
       │  Mirror Filter               │
       │  (Inbound/Outbound Rules)    │
       └──────── Mirror Tunnel ───────┘
                 (Inline Load Balancer 경유)
```

---

## 📝 시험 대비 체크리스트

- [ ] VPC: 논리적으로 격리된 고객 전용 사설 클라우드 네트워크
- [ ] VPC 적용 리전: **한국, 일본, 싱가포르**
- [ ] VPC 계정당 최대 생성 수: **3개**
- [ ] VPC IP 범위: RFC 1918 대역 (10.x, 172.16.x, 192.168.x), Netmask /28~/16
- [ ] VPC당 최대 Subnet 수: **200개**
- [ ] Subnet 종류: Public, Private, Load Balancer, Baremetal, NAT Gateway 전용
- [ ] VPC Peering: **1:1** 연결, 타 계정 VPC 연결도 가능
- [ ] Transit VPC: **1:N** 연결, 허브 역할
- [ ] Transit VPC: 메인 계정당 **최대 1개**
- [ ] Public Subnet: Internet Gateway 연결, 외부 접근 가능
- [ ] Private Subnet: 외부 접근 불가, NAT Gateway로 아웃바운드만
- [ ] NACL: Subnet 방화벽 (Stateless) / ACG: NIC 방화벽 (Stateful)
- [ ] NACL Deny-Allow Group: VPC당 최대 **4개**
- [ ] Packet Mirroring: 패킷 캡처 후 원하는 서버로 전송, 보안 강화 목적

---

## 🧠 암기 핵심 문장

> "**VPC는 계정당 3개, 한국·일본·싱가포르에서 사용 가능.**"  
> "**Transit VPC는 허브: 1:N 연결, 계정당 1개.**"  
> "**Peering은 1:1, Transit은 1:N.**"  
> "**NACL은 Stateless라 Inbound/Outbound 둘 다 설정해야 한다.**"

---

*다음 챕터: `15_로드밸런서.md` → 트래픽을 균등하게 나눠라*
