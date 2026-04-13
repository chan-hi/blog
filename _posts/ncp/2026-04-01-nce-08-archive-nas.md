---
title: "Chapter 8: 깊은 저장소와 공유 창고 — Archive Storage & NAS"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, archive-storage, nas, nfs, multipart-upload]
description: "NCP NCE 자격증 대비 - Archive Storage 비용 구조, NAS 스펙/프로토콜, 멀티파트 업로드 제한 완벽 정리"
---

> **학습 목표**: Archive Storage의 비용 구조, NAS의 스펙/프로토콜, 멀티파트 업로드 제한을 완벽히 암기한다.

---

## 이야기 1: 쌓여가는 오래된 데이터

클라우드빵집 운영 3년차. 오래된 주문 데이터가 수 테라바이트가 됐다.

> "법적으로 5년은 보관해야 하는데...  
> Object Storage에 두기엔 너무 비싸. 어떡하지?"

지수가 말했다.

> "**Archive Storage** 써. 저장 비용이 Object Storage보다 훨씬 싸거든.
> 대신 꺼낼 때 API 비용이 비싸니까, 자주 안 볼 데이터를 넣어야 해."

---

## 🔑 핵심 개념 0: Object Storage 기초 (Archive Storage 이해의 배경)

Archive Storage를 제대로 이해하려면 먼저 Object Storage의 스펙을 알아야 한다.

**Object Storage** = 인터넷 상에 원하는 데이터를 저장하고 사용할 수 있도록 구축된 오브젝트 기반의 무제한 파일 저장 스토리지.

### Object Storage 주요 스펙

| 항목 | 사양 |
|------|------|
| **생성 가능한 버킷 개수** | **1,000개 이하** |
| **객체명 길이** (폴더 경로 포함) | **1,024 Byte 이하** |
| **콘솔 업로드** 단일 파일 최대 | **1GB 이하** |
| **콘솔 다운로드** 단일 파일 최대 | **2GB 이하** |
| **업로드 API** (PutObject, UploadPart) | **10TB 이하** |
| **다운로드 API** (getObject) | **제한 없음** |

### Object Storage 특징

- **S3 Compatibility API** 지원
- **Data Lifecycle** 지원 (자동으로 Archive Storage로 이동)
- SubAccount와의 연동으로 접근 제어 가능
- ImageOptimizer, Cloud Hadoop, Cloud Log Analytics 등 NCP 내 다양한 상품과 통합/연계
- **정적 웹사이트 호스팅** 가능
- Linux에서 **s3fuse**, **goofys**를 이용해 NFS 마운트처럼 사용 가능 (단, 오픈소스 특성상 안정성 보장 없음, 속도 느림)

> 📌 **Object Storage vs Archive Storage의 핵심 차이**: Object Storage는 입출력 속도가 빠르므로 자주 사용하는 데이터에, Archive Storage는 장기 보관 데이터에 사용한다.

---

## 🔑 핵심 개념 1: Archive Storage

**Archive Storage** = 자주 꺼내지 않는 데이터를 **장기 보관**하는 저렴한 스토리지.  
데이터 아카이빙 및 장기 백업에 최적화된 스토리지 서비스.

### 비용 구조 (⭐ 시험 단골!)

| 항목 | Object Storage | **Archive Storage** |
|------|---------------|-------------------|
| **저장 비용** | 중간 | **저렴** ✅ |
| **API 처리 비용** | 중간 | **비쌈** ❌ |

> ⚠️ **핵심 공식**: "Archive = 저장은 싸다, 꺼낼 때 비싸다"  
> → 자주 꺼내지 않을 데이터를 넣어야 의미가 있음!

### Archive Storage 주요 특징

| 항목 | 내용 |
|------|------|
| **최소 보관 기간** | **없음** (타 클라우드: 90일/180일 제약 있는 경우도 있음) |
| **Lifecycle** | Object → Archive 자동 이동 지원 |
| **지원 API** | **S3 API** + **OpenStack swift API** 모두 지원 |
| **CLI 도구** | **OpenStack keystone/swift CLI** |
| **SubAccount 연동** | 권한 관리 기능 제공 |
| **데이터 보안** | 데이터 전송 시 **SSL** 지원 |
| **확장성** | **페타바이트(PB) 규모**까지 확장 가능 |
| **긴급 다운로드** | 아카이빙 데이터 **수 분 내** 다운로드 지원 |

> 📌 NCP Archive Storage는 최소 보관 기간이 **없다**!  
> (AWS Glacier 등은 최소 90일 보관 의무가 있음)

---

### Archive Storage 실무 활용

```
[Lifecycle 자동화 설정]

Object Storage (현재 파일)
       ↓ (30일 후 자동 이동)
Archive Storage (오래된 파일)
       ↓ (5년 후 자동 삭제)
삭제

→ 비용 최적화 완전 자동화!
```

```bash
# Archive Storage 제어 — OpenStack swift CLI
swift upload my-archive ./old-orders.tar.gz

# Archive Storage 파일 목록 확인
swift list my-archive
```

---

## 🔑 핵심 개념 2: 대용량 파일 멀티파트 업로드 (DLO/SLO)

Archive/Object Storage에 **5GB를 초과하는 파일**을 올리려면 멀티파트 업로드를 써야 한다.  
이를 **DLO(Dynamic Large Object)** / **SLO(Static Large Object)** 방식이라 부른다.

| 항목 | 내용 |
|------|------|
| **방식** | 파일을 Segment로 분할 후 **병렬 업로드** |
| **Segment 최대 크기** | **5GB** |
| **5GB 초과 파일** | **반드시 분할** 업로드 |
| **DLO** | Dynamic Large Object — 세그먼트를 동적으로 참조 |
| **SLO** | Static Large Object — 세그먼트 목록을 정적으로 명시 |

```
[멀티파트 업로드 동작]

대용량 파일 (예: 20GB)
       ↓ 분할
  Segment 1 (5GB) ──┐
  Segment 2 (5GB) ──┼──→ 병렬 업로드 → 서버에서 합치기
  Segment 3 (5GB) ──┤
  Segment 4 (5GB) ──┘

→ 빠르고 안정적! 중간에 실패해도 해당 Segment만 재업로드
```

> 📌 파일 하나의 크기가 너무 커서 업로드 속도가 느릴 경우에도 멀티파트 업로드로 속도 향상 가능

---

## 이야기 2: 직원들이 파일을 공유하려면?

클라우드빵집에 레시피 팀이 생겼다. 10명의 직원이 레시피 파일을 함께 편집한다.

> "서버 A에 파일 올리면 서버 B에서 못 보잖아.
> 어떻게 해야 여러 서버에서 같은 파일을 공유할 수 있어?"

지수가 답했다.

> "**NAS** 써. 여러 서버가 네트워크로 같은 스토리지를 마운트해서 쓸 수 있어."

---

## 🔑 핵심 개념 3: NAS (Network Attached Storage)

**NAS** = 여러 서버가 **네트워크를 통해 공유**하는 스토리지.

### NAS 기본 스펙

| 항목 | 내용 |
|------|------|
| **최소 용량** | **500GB** |
| **최대 용량** | **20TB** |
| **증설 단위** | **100GB** 단위 |
| **프로토콜** | **NFS** (Linux 계열) / **CIFS** (Windows 계열) |

### NAS 주요 특징

| 항목 | 내용 |
|------|------|
| 여러 서버 공유 | ✅ 가능 (핵심 기능) |
| 타 계정 서버 마운트 | ✅ ACL에 사설 IP 등록 시 가능 |
| 자동 스냅샷 | ✅ 지원 (볼륨 용량 안에서 스냅샷 이미지 생성 및 복구) |
| 이중화 구성 | ✅ 자체 RAID로 내구성 확보 |
| **동적 볼륨 사이즈 조정** | ✅ 가능 |
| **이벤트 알람** | ✅ 기능 제공 |

> ⚠️ **시험 포인트**  
> ACL 설정으로 **다른 계정의 서버**에서도 NAS를 마운트해서 사용 가능!

---

### Block Storage vs NAS 비교

| 항목 | **Block Storage** | **NAS** |
|------|-----------------|---------|
| 속도 | **빠름** | 약간 느림 (네트워크 경유) |
| 동시 접근 | **1대 서버만** | **여러 서버 공유** ✅ |
| 프로토콜 | iSCSI, NVMe-oF | NFS, CIFS |
| 최소 추가 용량 | **10GB** 단위 | **100GB** 단위 (최초 500GB) |
| 동적 볼륨 조정 | 제한적 | ✅ 가능 |
| 스냅샷 | - | ✅ 자동 스냅샷 |
| 이벤트 알람 | - | ✅ 제공 |
| 용도 | OS 부팅, DB | 공유 파일 저장소 |

---

## 🔑 핵심 개념 4: NAS 마운트 설정 (`/etc/fstab`)

서버를 재부팅해도 NAS가 자동 연결되게 하려면 `/etc/fstab`에 등록해야 한다.

### fstab 설정 형식

```
(1) NAS_IP:/볼륨명  (2) 마운트포인트  (3) nfs  (4) defaults  (5) 0  (6) 0
```

실제 예시:

```
10.10.10.10:/n0000000_volume1  /mnt/nas  nfs  defaults  0  0
```

### 각 필드 상세 설명

| 번호 | 항목 | 설명 |
|------|------|------|
| **(1)** | `10.00.00.00:/n000000_volume1` | 볼륨의 마운트 정보 (NAS IP + 볼륨명) |
| **(2)** | `/mnt/nas` | 서버의 마운트 포인트 |
| **(3)** | `nfs` | 파일 시스템 종류 |
| **(4)** | `defaults` | 마운트 옵션 |
| **(5)** | `0` | 덤프 설정 |
| **(6)** | `0` | fsck 검사 순서 |

### (3) 파일 시스템 종류 — OS별 차이 주의!

| OS | 파일 시스템 |
|----|-----------|
| CentOS 6.x, Ubuntu Server | `ext4` |
| CentOS 7.x, Rocky 8.x | `xfs` |
| NAS 마운트 | `nfs` (공통) |

### (4) `defaults` 옵션 — 5가지 세부 옵션 포함

| 옵션 | 기능 |
|------|------|
| `auto` | 부팅 시 **자동으로 마운트** |
| `rw` | **읽기와 쓰기** 가능하도록 마운트 |
| `nouser` | **root 계정만** 마운트 가능하도록 설정 |
| `exec` | **파일 실행** 허용 |
| `suid` | **SetUID와 SetGID** 허용 |

> 📌 `defaults` = 위 5가지 옵션을 **모두** 적용하는 편의 키워드!

### (5)(6) 덤프 & fsck 설정

| 값 | 덤프 설정 | fsck 검사 순서 |
|----|----------|--------------|
| `0` | 덤프되지 않는 파일 시스템 (NAS는 **0**) | 부팅 시 검사 안 함 (NAS는 **0**) |
| `1` | 덤프 가능한 파일 시스템 | 루트 파티션 먼저 검사 |

---

### NFS 버전 확인

NAS를 마운트한 후 `mount -v` 명령으로 실제 NFS 버전을 확인할 수 있다.

```bash
# NFS 버전 확인
mount -v
```

실제 출력 예시:

```
10.250.53.73:/n871882_nce on /nas type nfs4
(rw,relatime,vers=4.0,rsize=65536,wsize=65536,namlen=255,
 hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,
 clientaddr=10.36.5.153,local_lock=none,addr=10.250.53.73)
```

**CentOS 7.x 이상, Ubuntu 14.x 이상은 NFSv4로 동작**한다.

| NFS 옵션 | 값 | 의미 |
|---------|-----|------|
| `vers` | `4.0` | NFS 버전 4.0 |
| `rsize` | `65536` | 읽기 블록 크기 (64KB) |
| `wsize` | `65536` | 쓰기 블록 크기 (64KB) |
| `hard` | - | 서버 응답 없으면 계속 재시도 |
| `proto` | `tcp` | TCP 프로토콜 사용 |
| `timeo` | `600` | 타임아웃 값 |
| `retrans` | `2` | 재전송 횟수 |

---

### NAS 마운트 명령어 요약

```bash
# /etc/fstab에 등록 (부팅 시 자동 마운트)
echo "10.10.10.10:/n0000000_volume1  /mnt/nas  nfs  defaults  0  0" >> /etc/fstab

# fstab 설정 즉시 적용 (재부팅 없이)
mount -a

# 직접 즉시 마운트
mount -t nfs 10.10.10.10:/n0000000_volume1 /mnt/nas

# NFS 버전 및 마운트 옵션 확인
mount -v
```

> 📌 **2가지 등록 방법**:  
> 1. `/etc/fstab`에 등록 — 부팅 시 자동 마운트  
> 2. `/etc/rc.d/rc.local`에 등록 — 부팅 스크립트에 mount 명령 추가

---

## 전체 스토리지 선택 가이드

```
어떤 데이터인가요?

서버 OS 또는 DB 데이터 (빠른 I/O 필요)
  └→ Block Storage

여러 서버가 공유해야 하는 파일
  └→ NAS (NFS/CIFS)

이미지, 영상, 대용량 파일 (인터넷 접근 필요)
  └→ Object Storage (S3 API / AWS CLI)
     ※ 버킷 최대 1,000개, API 업로드 최대 10TB

자주 안 꺼내는 장기 보관 데이터
  └→ Archive Storage (OpenStack swift)
     ※ 최소 보관 기간 없음, PB 규모까지 확장
```

---

## 📝 시험 대비 체크리스트

- [ ] Object Storage 버킷 개수: **1,000개 이하**
- [ ] Object Storage 객체명 길이: **1,024 Byte 이하**
- [ ] Object Storage 콘솔 업로드: **1GB 이하** / 다운로드: **2GB 이하**
- [ ] Object Storage API 업로드 (PutObject, UploadPart): **10TB 이하**
- [ ] Object Storage API 다운로드 (getObject): **제한 없음**
- [ ] Archive: 저장 비용 **저렴**, API 처리 비용 **비쌈**
- [ ] Archive: 최소 보관 기간 **없음**
- [ ] Archive CLI: **OpenStack keystone/swift**
- [ ] Archive: S3 API + **swift API** 모두 지원
- [ ] Archive: 데이터 전송 시 **SSL** 지원
- [ ] Archive: **PB(페타바이트) 규모**까지 확장 가능
- [ ] Archive: 긴급 다운로드 **수 분 내** 지원
- [ ] 멀티파트 업로드: **DLO**(Dynamic Large Object) / **SLO**(Static Large Object)
- [ ] 멀티파트 업로드 Segment 최대: **5GB**
- [ ] 5GB 초과 파일 → 반드시 **분할** 업로드
- [ ] NAS 최소 용량: **500GB**
- [ ] NAS 최대 용량: **20TB**
- [ ] NAS 증설 단위: **100GB**
- [ ] NAS 프로토콜: **NFS**(Linux) / **CIFS**(Windows)
- [ ] NAS: 타 계정 서버도 **ACL 등록 시 마운트** 가능
- [ ] NAS: **동적 볼륨 사이즈 조정** 가능
- [ ] NAS: **이벤트 알람** 기능 제공
- [ ] fstab NAS: `defaults  0  0`
- [ ] `defaults` = auto + rw + nouser + exec + suid (**5가지** 옵션)
- [ ] fstab 파일 시스템: CentOS 6.x/Ubuntu → **ext4**, CentOS 7.x/Rocky 8.x → **xfs**
- [ ] CentOS 7.x 이상 / Ubuntu 14.x 이상 → **NFSv4**로 동작
- [ ] NFS 읽기/쓰기 블록 크기: **rsize=65536 / wsize=65536** (64KB)

---

## 🧠 암기 핵심 문장

> "**Archive = 저장 싸고 꺼낼 때 비싸다. 최소 보관 기간 없음. OpenStack swift. SSL 지원. PB 규모 확장.**"  
> "**NAS = 500GB~20TB, 100GB 증설, NFS(Linux)/CIFS(Windows), 여러 서버 공유, ACL로 타 계정도 가능.**"  
> "**멀티파트(DLO/SLO): Segment 최대 5GB. 5GB 초과 시 반드시 분할.**"  
> "**fstab defaults = auto+rw+nouser+exec+suid. 덤프·fsck는 둘 다 0. CentOS 7+ / Ubuntu 14+ → NFSv4.**"

---

*다음 챕터: `09_Backup전략.md` → 재해에 대비하라*
