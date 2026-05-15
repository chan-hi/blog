---
title: "[NCE] 05. 네트워크, 로드밸런서, 미디어"
date: 2026-05-14 09:40:00 +0900
categories: [NCP, NCE]
tags: [NCP, NCE, Load Balancer, NLB, ALB, DNS, CDN, GlobalEdge, LiveStation, VOD, NAT Gateway]
description: "OSI 7계층, Transit VPC, Load Balancer(NLB/NPLB/ALB) 비교, NAT Gateway, DNS, GlobalEdge CDN, GTM/GSLB, 미디어 서비스 정리"
---

## 핵심 요약

- Transit VPC: 계정당 최대 1개, 1:N 연결 / VPC Peering: 1:1 연결
- NLB: TCP/UDP, DSR 지원, Hash/RR 알고리즘만 / ALB: HTTP/HTTPS, URL 기반 분기, 고정 IP
- NPLB/ALB CPS: 30K/60K/90K/120K
- NAT Gateway: Zone당 5개
- DNS TTL 기본 300초, 최대 10개 레코드
- LiveStation: RTMP 입력, 최대 1만 동접, 채널당 최대 5개 변환

## OSI 7계층 / TCP/IP

| OSI Layer | 번호 | 데이터 단위 | 주요 프로토콜 |
|-----------|------|------------|--------------|
| Application | 7 | message | DHCP, FTP, DNS, HTTP, SMTP |
| Presentation | 6 | message | - |
| Session | 5 | message | - |
| Transport | 4 | Segment(TCP)/Datagram(UDP) | TCP, UDP |
| Network | 3 | packet | IPv4, IPv6, ARP |
| Data Link | 2 | frame | MAC Address (Ethernet) |
| Physical | 1 | Bit | Ethernet cable |

## CIDR / 서브넷 마스크

- IPv4: 총 32bit, 호스트 수 = `2^(32 - bitmask)`

| Bitmask | 호스트 수 |
|---------|----------|
| /25 | 128 |
| /26 | 64 |
| /27 | 32 |
| /28 | 16 |
| /29 | 8 |
| /30 | 4 |

## Transit VPC

- VPC와 VPC, VPC와 인터넷, VPC와 온프레미스 간 **중앙 허브 역할**
- **계정당 최대 1개** 생성 가능
- **1:N 연결** (VPC Peering은 1:1 연결)

## Load Balancer

### 구성 요소

| 요소 | 설명 |
|------|------|
| **Listener** | 클라이언트 요청 수신 후 적절한 Target Group으로 라우팅 |
| **Rule** | 대상 그룹으로 전달 여부 판단 기준 |
| **Target Group** | 부하 분산 대상들의 모임 (동일 VPC 내 서버만) |

- 헬스체크 주기: **5~300초**
- Target Group을 **다수의 LB에 연결 불가**

### 종류 비교

| 항목 | NLB | NPLB | ALB |
|------|-----|------|-----|
| **레이어** | L4 (TCP/UDP) | L4 Proxy | L7 (HTTP/HTTPS) |
| **DSR** | **지원** | 미지원 | 미지원 |
| **SSL 오프로드** | 미지원 | 지원 | 지원 |
| **URL 기반 분기** | 미지원 | 미지원 | **지원** |
| **HTTP/2** | 미지원 | 미지원 | **지원** |
| **알고리즘** | Hash, RR | RR, LC, Source Hash | RR, LC, Source Hash |
| **고정 세션** | 지원 | 미지원 | 지원 |
| **Multizone** | 미지원 | 지원 | 지원 |
| **CPS** | Dynamic Sizing | 30K~120K | 30K~120K |

#### NLB 특징
- TCP 레벨 고성능, **DSR(Direct Server Return)** 지원 → Client IP 그대로 로깅
- 알고리즘: **Hash, Round Robin만 제공**

#### ALB 특징
- **고정 IP 제공**
- URL/Host Header/Path Pattern 기반 분기, 가중치 기반 분기

### 로드밸런싱 알고리즘

| 알고리즘 | 설명 |
|----------|------|
| **Round Robin** | 순차 분배 |
| **Least Connection** | 연결이 가장 적은 서버로 분배 |
| **Source IP Hash** | 클라이언트 IP 해시로 고정 서버 매핑 |

### 주요 기능

- **유휴 제한 시간 (Idle Timeout)**: 일정 시간 요청 없으면 연결 종료, 리소스 절약
- **고정 세션 (Sticky Session)**: 항상 같은 서버로 전송, 로그인/장바구니 세션 유지
- **SSL Redirect**: 443 포트 수신 후 80 포트 서버로 전달
- **X-Forwarded-For**: Proxy 이용 시 클라이언트 IP 헤더

## NAT Gateway

- Private Subnet 내 자원의 **인터넷 Outbound 통신** 지원 (사설 IP → 공인 IP 변환)
- **Zone당 5개** 생성 가능

## DNS

### 레코드 타입

| 레코드 | 설명 |
|--------|------|
| **A** | 도메인 → IPv4 주소 |
| **AAAA** | 도메인 → IPv6 주소 |
| **CNAME** | 도메인 별명(alias). 다른 레코드와 공존 불가 |
| **MX** | 메일 수신 서버. Preference 작을수록 우선순위 높음 |
| **SPF** | 스팸 방지. 메일 발송 서버 IP 검증 |
| **TXT** | 도메인에 텍스트 정보 추가 |
| **SRV** | 서비스 호스팅 서버의 위치(호스트명+포트) |
| **CAA** | 인증서 발급 CA 확인, 가짜 인증서 방지 |

- **TTL 기본 300초**, 최대 7일
- **최대 10개** 레코드 추가 가능

## IPsec VPN

- 고객 사내망과 NCP 간 사설 통신
- **Active-Standby 방식 이중화** 구성
- 설정 순서: VGW 생성 → IPsec VPN Gateway/Tunnel 생성 → Route Table 설정

## GlobalEdge (CDN)

- 다양한 원본 및 백엔드 지원
- **압축**: gzip, brotli 지원
- **프로토콜**: HTTP/2, TLSv1.3
- **인증**: Signed URL (Secure Token), JWT, Media Vault
- **접근 제어**: IP/GEO/Referer 헤더 기반

## GTM / GSLB

| 서비스 | 설명 |
|--------|------|
| **GTM** (Global Traffic Manager) | DNS 기반 트래픽 분산. Round Robin / Weighted / Geolocation / CIDR 알고리즘 |
| **GSLB** (Global Server Load Balancing) | DNS 기반 글로벌 분산. RTT 기반 응답, GeoIP 기반 연결 |

## 미디어 서비스

### Image Optimizer
- 이미지 **Object Storage에 저장** → Image Optimizer 변환 → **GlobalEdge** 통해 배포
- 쿼리 스트링으로 변환 룰 작성
- 리사이즈, 크롭, 워터마크, 자동 회전, 얼굴 인식 제공

### VOD Station
- VOD 동영상을 다양한 환경에 맞게 변환하여 송출
- **Multi DRM**: **Widevine, FairPlay, PlayReady** 지원
- 출력 포맷: HLS, DASH
- 다중 출력 최대 **5개**, 썸네일 최대 **10개**

### LiveStation
- 실시간 방송 플랫폼
- **입력 프로토콜**: RTMP
- **출력**: HLS, LL-HLS, MPEG-DASH, RTMP
- CDN 연동 시 **최대 1만 명 동접**
- **채널당 최대 5개** 동시 변환
- 기능: TimeShift, DVR, Re-Stream, VOD2LIVE

### Media Connect Center
- Object Storage + VOD Station + LiveStation **통합 관리 서비스**

## 시험 암기 포인트

- **Transit VPC**: 계정당 1개, 1:N 연결 / **VPC Peering**: 1:1 연결
- **NLB**: TCP/UDP, DSR 지원, Hash/RR만 / **ALB**: HTTP/HTTPS, URL 분기, 고정 IP
- **NPLB/ALB CPS**: 30K/60K/90K/120K
- **NAT Gateway**: Zone당 5개
- **DNS TTL**: 기본 300초, 최대 레코드 10개
- **LiveStation**: RTMP 입력, 동접 1만, 채널당 5개 변환
- **VOD Multi DRM**: Widevine + FairPlay + PlayReady
- **Image Optimizer**: Object Storage + GlobalEdge 연동

