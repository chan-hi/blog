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

## 🔑 핵심 개념 1: Archive Storage

**Archive Storage** = 자주 꺼내지 않는 데이터를 **장기 보관**하는 저렴한 스토리지.

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

## 🔑 핵심 개념 2: 대용량 파일 멀티파트 업로드

Archive/Object Storage에 **5GB를 초과하는 파일**을 올리려면 멀티파트 업로드를 써야 한다.

| 항목 | 내용 |
|------|------|
| **방식** | 파일을 Segment로 분할 후 **병렬 업로드** |
| **Segment 최대 크기** | **5GB** |
| **5GB 초과 파일** | **반드시 분할** 업로드 |

```
[멀티파트 업로드 동작]

대용량 파일 (예: 20GB)
       ↓ 분할
  Segment 1 (5GB) ──┐
  Segment 2 (5GB) ──┼──→ 병렬 업로드 → 서버에서 합치기
  Segment 3 (5GB) ──┘
  Segment 4 (5GB) ──┘

→ 빠르고 안정적! 중간에 실패해도 해당 Segment만 재업로드
```

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
| 자동 스냅샷 | ✅ 지원 |
| 이중화 구성 | ✅ 자체 RAID로 내구성 확보 |

> ⚠️ **시험 포인트**  
> ACL 설정으로 **다른 계정의 서버**에서도 NAS를 마운트해서 사용 가능!

---

### Block Storage vs NAS 비교

| 항목 | **Block Storage** | **NAS** |
|------|-----------------|---------|
| 속도 | **빠름** | 약간 느림 (네트워크 경유) |
| 동시 접근 | **1대 서버만** | **여러 서버 공유** |
| 프로토콜 | iSCSI, NVMe-oF | NFS, CIFS |
| 용도 | OS 부팅, DB | 공유 파일 저장소 |

---

## 🔑 핵심 개념 4: NAS 마운트 설정 (`/etc/fstab`)

서버를 재부팅해도 NAS가 자동 연결되게 하려면 `/etc/fstab`에 등록해야 한다.

```bash
# /etc/fstab 설정 형식
[NAS IP]:/[볼륨명]  /[마운트 경로]  nfs  defaults  0  0

# 실제 예시
192.168.10.100:/vol01  /data/shared  nfs  defaults  0  0
```

| 항목 | 설명 |
|------|------|
| `defaults` | auto(부팅 시 자동 마운트) + rw + exec 등 5가지 기본 옵션 포함 |
| `0` (덤프 필드) | 백업 스케줄러(dump) 사용 여부 → NAS는 **0** (사용 안 함) |
| `0` (fsck 필드) | 부팅 시 디스크 검사 순서 → NAS는 **0** (검사 안 함) |

```bash
# NAS 마운트 명령 (즉시 마운트)
mount -t nfs 192.168.10.100:/vol01 /data/shared

# fstab 설정 테스트 (재부팅 없이 fstab 적용)
mount -a
```

---

## 전체 스토리지 선택 가이드

```
어떤 데이터인가요?

서버 OS 또는 DB 데이터 (빠른 I/O 필요)
  └→ Block Storage

여러 서버가 공유해야 하는 파일
  └→ NAS (NFS/CIFS)

이미지, 영상, 대용량 파일 (인터넷 접근 필요)
  └→ Object Storage (AWS CLI)

자주 안 꺼내는 장기 보관 데이터
  └→ Archive Storage (OpenStack swift)
```

---

## 📝 시험 대비 체크리스트

- [ ] Archive: 저장 비용 **저렴**, API 처리 비용 **비쌈**
- [ ] Archive: 최소 보관 기간 **없음**
- [ ] Archive CLI: **OpenStack keystone/swift**
- [ ] Archive: S3 API + **swift API** 모두 지원
- [ ] 멀티파트 업로드 Segment 최대: **5GB**
- [ ] 5GB 초과 파일 → 반드시 **분할** 업로드
- [ ] NAS 최소 용량: **500GB**
- [ ] NAS 최대 용량: **20TB**
- [ ] NAS 증설 단위: **100GB**
- [ ] NAS 프로토콜: **NFS**(Linux) / **CIFS**(Windows)
- [ ] NAS: 타 계정 서버도 **ACL 등록 시 마운트** 가능
- [ ] fstab NAS: `defaults  0  0`

---

## 🧠 암기 핵심 문장

> "**Archive = 저장 싸고 꺼낼 때 비싸다. 최소 보관 기간 없음. OpenStack swift.**"  
> "**NAS = 500GB~20TB, NFS(Linux)/CIFS(Windows), 여러 서버 공유.**"  
> "**멀티파트 Segment 최대 5GB.**"

---

*다음 챕터: `09_Backup전략.md` → 재해에 대비하라*
