---
title: "Chapter 13: 빵집의 도로망 — 네트워크 기초"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, network, osi, tcp-ip, cidr, subnet]
description: "NCP NCE 자격증 대비 학습 자료 - OSI 7계층, TCP/IP 4계층, CIDR 계산 기초"
---
> **학습 목표**: OSI 7계층, TCP/IP 모델, CIDR/서브넷 계산, VPC 개념 및 서브넷 구성을 완벽히 이해한다.

---

## 이야기: 택배 기사가 설명해주는 인터넷

클라우드빵집이 전국 배달을 시작했다.
민준은 문득 궁금해졌다.

> "내가 서버에서 보낸 데이터가 어떻게 고객 브라우저에 도달하지?"

베테랑 네트워크 엔지니어 재현이 설명했다.

> "택배 시스템이랑 똑같아. 포장하고, 배송 정보 붙이고, 물류 센터 거치고, 배달되는 것처럼.
> 네트워크도 계층(Layer)별로 역할이 나뉘어 있어."

---

## 핵심 개념 1: OSI 7계층 vs TCP/IP 모델

### OSI 7계층 (이론 모델)

OSI 기본 참조 모델은 네트워크 통신을 7개의 계층으로 나눠 설명하는 이론 모델이다. 각 계층마다 고유한 데이터 단위(PDU)와 역할이 있다.

| 계층 번호 | 계층 이름 | 데이터 단위 | 대표 프로토콜/장비 |
|---------|---------|-----------|-----------------|
| 7 | **응용 (Application)** | Message | HTTP, FTP, DNS, DHCP, SMTP, POP |
| 6 | **표현 (Presentation)** | Message | SSL/TLS, 암호화 |
| 5 | **세션 (Session)** | Message | 세션 연결/종료 |
| 4 | **전송 (Transport)** | Segment(TCP) / Datagram(UDP) | TCP, UDP |
| 3 | **네트워크 (Network)** | Packet | IP(IPv4, IPv6), ARP, RARP |
| 2 | **데이터링크 (Data Link)** | Frame | MAC Address, 이더넷, 토큰링 |
| 1 | **물리 (Physical)** | Bit | 이더넷 케이블, 와이어, NIC |

- **물리 계층**: 물리적인 인터페이스 (이더넷, 토큰링, MAC Address)
- **네트워크 계층**: IP 주소 기반 라우팅 (IPv4, IPv6, ARP, RARP)
- **전송 계층**: 포트 기반 데이터 전달 (TCP, UDP)
- **응용 계층**: 사용자와 맞닿는 프로토콜 (DHCP, FTP, DNS, HTTP, POP, SMTP)

### TCP/IP 4계층 (실무 모델)

실제 인터넷에서 사용하는 모델. OSI 7계층을 4개로 압축한 것이다.

| TCP/IP 계층 | 대응 OSI 계층 | 데이터 단위 | 주요 프로토콜 |
|-----------|------------|-----------|------------|
| **Application Layer** | 7 + 6 + 5 | **Message** | HTTP, FTP, DNS, DHCP, SMTP, POP |
| **Transport Layer** | 4 | **Segment(TCP) / Datagram(UDP)** | TCP, UDP |
| **Internet Layer** | 3 | **Packet** | IPv4, IPv6, ARP, RARP |
| **Network Layer** | 2 + 1 | **Frame / Bit** | 이더넷 케이블, MAC Address |

> **암기법**: "앱-트랜스-인터넷-네트워크" (A-T-I-N)  
> 데이터 단위 흐름: Message → Segment/Datagram → Packet → Frame → Bit

---

## 핵심 개념 2: TCP vs UDP

| 항목 | **TCP** | **UDP** |
|------|---------|---------|
| **연결 방식** | 연결 지향 (3-way handshake) | 비연결형 |
| **신뢰성** | 높음 (재전송, 순서 보장) | 낮음 (재전송 없음) |
| **속도** | 느림 | **빠름** |
| **용도** | 웹, 파일 전송, 이메일 | 스트리밍, DNS, 게임 |
| **데이터 단위** | Segment | Datagram |

---

## 핵심 개념 3: IP 주소와 CIDR

### IPv4 기본 구조

IPv4는 총 32bit(4byte)로 구성된 주소 체계다. 기본적으로 `X.X.X.X` 형태로 보이는 주소는 `8bit + 8bit + 8bit + 8bit`의 집합체다.

```
IPv4: 32bit = 4 byte = 4개 옥텟
예: 192.168.1.100
     ↑옥텟1 ↑옥텟2 ↑옥텟3 ↑옥텟4
```

### Classful Addressing (전통 방식)

인터넷상의 IP 주소를 규격화된 크기로 구분하는 방식. IP 주소를 클래스별로 규격화한다.

| 클래스 | 상위 비트 패턴 | IP 주소 범위 | 서브넷 마스크 | 네트워크 비트 | 호스트 비트 | 호스트 수 |
|-------|------------|-----------|------------|-----------|---------|---------|
| **A** | 0xxxxxxx | 1.0.0.0 ~ 127.255.255.255 | 255.0.0.0 (/8) | 8비트 | 24비트 | 약 1,600만 개 |
| **B** | 10xxxxxx | 128.0.0.0 ~ 191.255.255.255 | 255.255.0.0 (/16) | 16비트 | 16비트 | 약 65,000개 |
| **C** | 110xxxxx | 192.0.0.0 ~ 223.255.255.255 | 255.255.255.0 (/24) | 24비트 | 8비트 | 254개 |
| **D** | 1110xxxx | 224.0.0.0 ~ 239.255.255.255 | — | — | — | 멀티캐스트용 예약 |
| **E** | 1111xxxx | 240.0.0.0 ~ 255.255.255.255 | — | — | — | 연구·개발 목적 예약 |

### CIDR (Classless Inter-Domain Routing)

클래스 구분 없이 **비트(bit) 단위**로 주소를 부여하는 현대적 체계. 네트워크 사용 규모에 따라 적절한 서브넷팅을 통해 브로드캐스트 영역을 나눔과 동시에 IP 주소 낭비를 방지한다.

```
표기법: IP주소/비트마스크
예: 192.168.0.0/24

/24 = 앞 24bit가 네트워크 주소
      뒤 8bit가 호스트 주소
      → 호스트 수 = 2^(32-24) = 2^8 = 256개
```

### 호스트 수 계산 공식 (시험 단골!)

```
호스트 수 = 2^(32 - 비트마스크)

IP 대역 계산 예: 192.168.0.0/24 = 192.168.0.0 ~ (192.168.0.0 + 호스트 수 - 1)
                                 = 192.168.0.0 ~ 192.168.0.255
```

주요 비트마스크별 서브넷 마스크와 호스트 수:

| 비트마스크 | 서브넷 마스크 | 호스트 수 |
|---------|------------|---------|
| /25 | 255.255.255.128 | 128 |
| /26 | 255.255.255.192 | 64 |
| /27 | 255.255.255.224 | 32 |
| /28 | 255.255.255.240 | 16 |
| /29 | 255.255.255.248 | 8 |
| /30 | 255.255.255.252 | 4 |
| /31 | 255.255.255.254 | 2 |
| /32 | 255.255.255.255 | 1 |

---

## 핵심 개념 4: 사설 IP 대역 (RFC 1918)

인터넷에 직접 노출되지 않는 내부 네트워크 전용 IP 대역. RFC 1918에서 정의한다.

| 클래스 | 사설 IP 대역 | 주로 사용 |
|-------|-----------|---------|
| A | **10.0.0.0/8** | 대기업 내부망 |
| B | **172.16.0.0/12** | 중형 내부망 |
| C | **192.168.0.0/16** | 가정/소형 네트워크 |

> NCP VPC 생성 시 이 RFC 1918 사설 IP 대역(10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) 중에서 선택한다.

---

## 핵심 개념 5: VPC (Virtual Private Cloud)

이론을 배웠으니 이제 NCP에서 이를 어떻게 활용하는지 살펴보자.

> "그럼 NCP에서 우리만의 네트워크 공간은 어떻게 만들어?"
> 민준이 물었다.
>
> "VPC를 쓰면 돼. 클라우드 위에 우리만의 사설 도로망을 놓는 거야."
> 재현이 답했다.

### VPC 개념

VPC(Virtual Private Cloud)는 클라우드 상에서 **논리적으로 격리된 고객 전용 네트워크 공간**이다.

**주요 특징**:
- **전용 네트워크 사용**: 다른 네트워크와 상호 간섭 없이 논리적으로 완전히 분리된 네트워크
- **다양한 네트워크 토폴로지**: VPC 내부에 Public Subnet 또는 Private Subnet을 생성하여 고객 맞춤형 네트워크 환경 구성
- **강력한 보안**: ACG(Access Control Group)와 Network ACL(Network Access Control List)을 통해 네트워크 접근 제어
  - ACG: 서버 단계의 접근 제어
  - Network ACL: Subnet 단계의 접근 제어
- **VPC 간 내부 통신**: VPC Peering을 사용해 다른 VPC와 공인 IP 없이 내부 네트워크로 통신 가능

**제약 사항**:
- 현재 **한국, 일본, 싱가포르 리전**에 적용
- **계정당 최대 3개의 VPC** 생성 가능
- IP 주소 범위: RFC 1918 대역인 `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` 중에서 선택
- Netmask: **최소 /28에서 최대 /16**까지 생성 가능

### VPC Peering

VPC 간 연결을 위한 네트워크 구성 방식.

- 내 VPC 간 연결뿐만 아니라 **다른 계정과의 VPC 연결**도 가능
- 타 계정 VPC 연결 시 **로그인 ID, VPC ID, VPC 명** 입력 필요
- **1:1 연결 구조** (Transit VPC는 1:N 연결 구조와 비교됨)

---

## 핵심 개념 6: Subnet

VPC 안을 더 잘게 나누는 단위가 서브넷이다.

### Subnet 속성

- VPC 주소 범위 내에서 **CIDR 형태**로 주소 범위 지정
- **Zone을 지정**할 수 있으며, 동일한 Zone 내에 여러 Subnet 생성 가능
- VPC당 **최대 200개의 Subnet** 생성 가능
- **Public Subnet**: 외부 인터넷 통신이 가능한 서브넷. 이 안에 생성된 서버만 Public IP 부여 가능
- **Private Subnet**: 외부 인터넷 통신이 불가능한 서브넷. 인터넷 접속 시 NAT Gateway 별도 생성 필요 (Outbound 통신만 가능)

### Subnet 종류

| 종류 | 설명 |
|-----|-----|
| Public Subnet | 인터넷 게이트웨이(IGW) 연결, 외부 통신 가능 |
| Private Subnet | 내부 통신 전용, 인터넷 접속 시 NAT Gateway 필요 |
| Load Balancer 전용 Subnet | 로드밸런서 배치 전용 |
| Baremetal 전용 Subnet | 베어메탈 서버 전용 |
| NAT Gateway 전용 Subnet | NAT 게이트웨이 전용 |

### Internet Gateway 전용 여부

Subnet 생성 시 인터넷 연결 여부를 선택한다:
- **Y (Public)** 선택: Public Subnet 생성 → 서버에 공인 IP 부여 후 인터넷 통신 가능
- **N (Private)** 선택: Private Subnet 생성 → 내부 통신만 가능, 인터넷 접속 시 NAT Gateway 별도 생성 필요

### Subnet 아키텍처 패턴

```
[Public Subnet만 구성]                [Public + Private Subnet 구성]

인터넷                                 인터넷
  |                                       |
Route Table                          Route Table + NAT Gateway
  |                                    |              |
Public Subnet (10.0.0.0/24)       Public Subnet   Private Subnet
서버 (Public IP 보유)             (10.0.0.0/24)   (10.0.1.0/24)
                                  서버(공인IP)      서버(사설IP만)
```

---

## 핵심 개념 7: NACL (Network ACL)

- **Subnet 단위의 방화벽 서비스**
- Inbound/Outbound 트래픽의 허용/차단 규칙 모두 설정, 기본적으로 허용 규칙 적용
- **Stateless 방식**: 트래픽 상태를 저장하지 않아 Inbound 규칙과 Outbound 규칙을 별도로 설정해야 함
- 트래픽 허용 여부 결정 시 **규칙 우선순위** 순으로 처리
- 대상 Subnet 내 **모든 서버에 자동 적용** (ACG를 지정하는 사용자에게 의존할 필요 없음)
- **Deny-Allow Group**: 다수의 IP 그룹으로, Network ACL의 Inbound/Outbound 규칙 설정 시 접근 소스 또는 목적지 대상으로 활용. VPC 1개당 최대 4개 생성 가능

---

## Transit VPC

여러 VPC와 온프레미스를 연결하는 허브 역할이 필요할 때 Transit VPC를 사용한다.

- VPC와 VPC, VPC와 인터넷, VPC와 온프레미스 간 네트워크 상호 연결을 위한 **중앙 허브 역할**
- Endpoint Route, Transit VPC Connect와 연계하여 라우팅을 통해 연결(흐름) 제어 가능
- **1:N 연결 구조** (VPC Peering의 1:1과 다름)
- **메인 계정당 최대 1개** 생성 가능

---

## 실습: 네트워크 환경 구성 (Lab 1)

NCE 교재의 Lab 1 실습 구성 예시:

**VPC 생성**
- 이름: `lab1-vpc`
- IP 대역: `10.0.0.0/16`

**NACL 생성**
- 이름: `lab1-vpc-web-nacl`

**Public Subnet 생성**
- 이름: `lab1-vpc-web-subnet`
- IP 대역: `10.0.1.0/24`

---

## 시험 대비 체크리스트

- [ ] OSI 7계층 순서: 물리→데이터링크→네트워크→전송→세션→표현→응용
- [ ] TCP/IP 데이터 단위: Message→Segment/Datagram→Packet→Frame→Bit
- [ ] Transport Layer: TCP(Segment), UDP(Datagram)
- [ ] Internet Layer: IPv4, IPv6, ARP, RARP → Packet 단위
- [ ] TCP: 연결 지향, 신뢰성 높음 / UDP: 비연결, 빠름
- [ ] IPv4: 32bit = 4byte 구조
- [ ] 호스트 수 = **2^(32-비트마스크)**
- [ ] 사설 IP (RFC 1918): 10.x.x.x / 172.16-31.x.x / 192.168.x.x
- [ ] VPC: 논리적으로 격리된 고객 전용 네트워크 공간
- [ ] VPC 생성 가능 리전: **한국, 일본, 싱가포르**
- [ ] VPC: 계정당 **최대 3개** 생성 가능
- [ ] VPC IP 대역: RFC 1918 대역, Netmask /28 ~ /16
- [ ] Subnet: VPC당 최대 200개
- [ ] Public Subnet: 공인 IP 부여 가능 / Private Subnet: NAT Gateway 필요
- [ ] NACL: Subnet 단위, Stateless 방화벽
- [ ] Transit VPC: 1:N 연결 허브, 계정당 최대 1개
- [ ] VPC Peering: 1:1 연결

---

## 암기 핵심 문장

> "**OSI는 7계층: 물데네전세표응 / TCP/IP는 4계층: 네인트앱**"  
> "**호스트 수 = 2^(32-마스크)**"  
> "**VPC = 논리적 격리 네트워크, 계정당 3개, 한국·일본·싱가포르**"  
> "**NACL = Subnet 방화벽, Stateless / ACG = 서버 방화벽**"

---

*다음 챕터: `14_VPC_TransitVPC.md` → NCP 네트워크의 기초 설계*
