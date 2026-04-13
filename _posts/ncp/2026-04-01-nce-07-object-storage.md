---
title: "Chapter 7: 무한 창고 — Object Storage 심화"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, object-storage, s3, aws-cli, lifecycle]
description: "NCP NCE 자격증 대비 - Object Storage 제한 사항 수치, 권한 관리, AWS CLI 사용 이유 완벽 정리"
---

> **학습 목표**: Object Storage의 제한 사항 수치, 권한 관리, AWS CLI 사용 이유를 완벽히 암기한다.

---

## 이야기: 수십만 장의 이미지를 어디에?

클라우드빵집의 상품 이미지가 수십만 장에 달했다.
민준은 고민했다.

> "이 많은 이미지를 서버 디스크에 다 저장하기엔 너무 비싸.
> CDN으로도 연동하고 싶고, 웹에서도 직접 URL로 접근하게 하고 싶어."

지수가 답했다.

> "Object Storage야. S3 호환이라서 AWS CLI로도 쓸 수 있어.
> URL로 직접 접근 가능하고, 정적 웹사이트 호스팅도 되거든."

민준이 의아한 표정을 지었다.

> "근데 Object Storage가 뭐가 다른 거야? 그냥 큰 하드디스크 아닌가?"

지수는 화이트보드에 그림을 그리기 시작했다.

---

## 배경 지식: 파일 시스템의 진화

Object Storage를 이해하려면 파일 시스템이 어떻게 발전해왔는지 알아야 한다.

### 로컬 파일 시스템의 한계

일반적인 로컬 파일 시스템은 로컬에 장착된 하드디스크(고정된 크기의 섹터와 블록으로 저장)에 맞춰 설계된 시스템이다. 파일을 저장하는 방법에는 두 가지가 있다.

**연결 할당 방식 (링크 구조)**  
파일이나 디렉토리의 시작 블록과 끝 블록만 가지고 있으면서, 각 블록은 자신의 다음 블록을 가리키는 포인터를 지닌다. 문제는 파일 전체를 읽으려면 Sequential Search가 필요해서 느리고, 데이터가 아닌 포인터 정보를 저장하느라 메모리를 낭비한다는 점이다.

**색인 할당 방식 (색인 구조)**  
Index Block에 해당 파일이 차지하는 블록들의 리스트를 저장한다. 덕분에 파일의 Random Access를 구현할 때 모든 블록을 sequential하게 읽어야 했던 단점이 해결된다. 최근의 많은 파일 시스템이 이 방식을 사용한다.

그러나 로컬 파일 시스템은 세 가지 근본적인 한계가 있다.

- **용량의 한계**: 물리적 디스크 크기에 종속
- **장애 복구의 한계**: 디스크 고장 시 데이터 유실
- **접근 경로 제한**: 로컬에서만 접근 가능

### 네트워크 파일 시스템 (NFS, CIFS)

NFS(Network File System)는 네트워크에 파일을 저장하는 메커니즘이다. 사용자가 원격 컴퓨터에 있는 파일 및 디렉토리에 접근하고, 해당 파일들이 로컬에 있는 것처럼 처리할 수 있도록 허용하는 분산 파일 시스템이다. 하지만 용량을 높이려면 고가의 인프라 비용이 발생한다.

### 분산 파일 시스템 (HDFS 등)

분산 파일 시스템이란 네트워크를 통해 물리적으로 다른 위치에 있는 여러 컴퓨터에 자료를 분산 저장하여, 마치 로컬 시스템에서 사용하는 것처럼 동작하게 하는 시스템이다. HDFS처럼 파일을 쪼개서 여러 노드에 분산 및 복제하는 방식을 사용한다. 용량 한계를 극복하고, 장애 시에도 복구가 가능하며, 다양한 접근 경로를 제공한다.

### Object Storage — 분산 파일 시스템의 또 다른 형태

Object Storage도 분산 파일 시스템의 일종이다. HDFS와 다른 점은 **파일을 쪼개지 않고 복제**한다는 것이다.

```
[Object Storage 내부 구조]

메타데이터 서버 (MDS)
- Owner(홍길동) → 복제본 1,2,3
- Owner(이몽룡) → 복제본 3,4,5
- Owner(성춘향) → 복제본 2,5,6

    ↓ 조회

데이터 서버 (DS1, DS2, DS3, DS4, DS5, DS6)
- 실제 파일 데이터를 분산 저장
```

분산 파일 시스템의 대표적인 종류로는 아마존 S3, HDFS, AFS(Andrew File System), DCE 분산 파일 시스템, Coda, Tahoe-LAFS 등이 있다.

---

## 핵심 개념 1: Object Storage 기본 특징

> "그래서 Object Storage가 뭐야?" 민준이 물었다.  
> "인터넷 위의 무한 창고." 지수가 답했다.

Object Storage는 인터넷상에 원하는 데이터를 저장하고 사용할 수 있도록 구축된 **객체 기반의 무제한 파일 저장 스토리지**다.

| 항목 | 내용 |
|------|------|
| **API 호환** | **AWS S3 API** 호환 (S3 Compatibility API 지원) |
| **CLI 도구** | **AWS CLI** (NCP 자체 CLI 아님!) |
| **관리 방법** | 콘솔, RESTful API, SDK 등 다양한 방법 지원 |
| **URL 접근** | 저장된 파일마다 **고유한 접근 URL** 부여 — 인터넷에서 직접 접근 가능 |
| **Lifecycle** | 데이터 자동 이동/삭제 주기 설정 가능 (Data Lifecycle 지원) |
| **정적 웹 호스팅** | 가능 (HTML, CSS, JS, 이미지 직접 서빙) |
| **데이터 복구** | 삭제 시 복구 불가 (정기 백업 권장) |
| **Sub Account 연동** | Sub Account와의 연동으로 접근 제어 가능 |
| **NCP 서비스 연계** | Image Optimizer, Cloud Hadoop, Cloud Log Analytics 등과 통합/연계 지원 |

> **시험 포인트**  
> Object Storage CLI는 **AWS CLI**다! (S3 호환이기 때문)  
> `aws s3 cp`, `aws s3 ls` 등의 명령어 사용

```bash
# AWS CLI로 Object Storage 사용 예시
aws s3 cp ./photo.jpg s3://my-bucket/ \
    --endpoint-url=https://kr.object.ncloudstorage.com

aws s3 ls s3://my-bucket/ \
    --endpoint-url=https://kr.object.ncloudstorage.com
```

> **참고**: S3가 아닌 Object Storage의 경우 `--endpoint-url` 옵션으로 저장 위치(endpoint)를 지정해야 한다.

---

## 핵심 개념 2: 주요 제한 사항 (필수 암기!)

숫자를 정확히 외워야 한다. 시험에 거의 매번 출제된다.

| 항목 | **제한 사양** |
|------|-------------|
| 생성 가능한 **버킷 개수** | 계정당 **1,000개** 이하 |
| **객체명 길이** (폴더 경로 포함) | **1,024 Byte** 이하 |
| **콘솔 업로드** 최대 크기 | 단일 파일 **1GB** 이하 |
| **콘솔 다운로드** 최대 크기 | **2GB** 이하 |
| **API 업로드** 최대 크기 | 단일 파일 **10TB** 이하 (PutObject, UploadPart) |
| **API 다운로드** 최대 크기 | **제한 없음** (getObject) |
| **버킷 저장 용량** | **제한 없음** |

> **시험 단골 3문제**  
> Q1: "콘솔로 올릴 수 있는 최대 파일 크기?" → **1GB**  
> Q2: "API로 올릴 수 있는 최대 파일 크기?" → **10TB**  
> Q3: "버킷은 계정당 최대 몇 개?" → **1,000개**

---

## 핵심 개념 3: Linux에서 NFS처럼 마운트하기

Object Storage를 일반 파일 시스템처럼 사용하고 싶다면?

Linux의 경우 **s3fuse** 또는 **goofys**를 이용하여 NFS 마운트하듯이 Object Storage를 마운트해서 사용할 수 있다. 단, 주의할 점이 있다.

- 두 도구 모두 **오픈소스**이므로 NCP에서 공식적인 안정성을 보장하지 않는다
- 일반 파일 시스템 접근보다 **속도가 느리다**

> 마운트 편의성은 있지만 프로덕션 환경에서는 직접 API 호출 방식이 더 안정적이다.

---

## 핵심 개념 4: 권한 관리 — 공개 관리와 ACL

Object Storage의 접근 제어는 두 가지 방식이 있다.

| 구분 | 대상 | 설명 |
|------|------|------|
| **공개 관리 (Public)** | 인터넷 전체 (NCP 외부) | URL로 누구나 접근 가능하게 공개하는 기능 |
| **권한 관리 (ACL)** | NCP 내부 사용자 | Sub Account 등에게 세밀한 권한 부여하는 기능 |

### 공개 관리 상세

- **버킷 공개**: 누구나 버킷 안에 있는 객체(파일/폴더) 목록을 확인하고 파일 업로드 가능
- **파일 공개**: 누구나 파일을 열어보고 다운로드 가능

### 권한 관리 상세 (ACL)

**버킷 단위로 부여할 수 있는 권한:**
- 버킷에 대한 목록 조회
- 파일 업로드
- ACL(Access Control List) 조회
- ACL 수정

**파일 단위로 부여할 수 있는 권한:**
- 파일 다운로드
- ACL 조회
- ACL 수정

```
[접근 제어 예시]

기본 상태 (비공개):
  외부 사용자 → 버킷 접근 불가

공개 설정:
  버킷 전체 공개 → 누구나 URL로 이미지 다운로드 가능
  특정 파일만 공개 → 해당 파일만 공개

권한 관리 (ACL):
  Sub Account A → my-bucket 목록 조회 + 파일 업로드 권한
  Sub Account B → my-bucket 파일 다운로드 권한만
  다른 계정 → 접근 불가
```

---

## 핵심 개념 5: Lifecycle (수명 주기 관리)

데이터를 자동으로 관리하는 강력한 기능이다.

모든 데이터는 시간이 지남에 따라 사용 빈도가 낮아진다. 활용도가 높은 데이터는 입출력 속도가 빠른 스토리지에 저장하고, 규제(Compliance) 대응과 향후 분석을 위해 장기간 저장이 필요한 데이터는 요금이 저렴한 스토리지를 이용하면 TCO 절감을 실현할 수 있다.

NCP Object Storage는 Archive Storage 대비 입출력 속도가 빠르므로, 자주 사용하는 데이터는 Object Storage에 저장하고, 장기 보관용 데이터는 Archive Storage에 저장할 수 있도록 **Lifecycle Management** 기능을 제공한다.

```
[Lifecycle 활용 시나리오]

Object Storage에 파일 업로드
       ↓
30일 경과 → Archive Storage로 자동 이동 (스케줄 기반 데이터 이동)
       ↓
1년 경과 → 자동 삭제

→ 비용 최적화 자동화!
```

> 자주 쓰는 파일은 Object Storage에,  
> 오래된 파일은 Lifecycle으로 자동으로 Archive Storage로 이동시켜 비용 절감.

---

## 핵심 개념 6: 정적 웹사이트 호스팅

Object Storage 하나만으로 **웹사이트를 운영**할 수 있다.

```
[정적 웹 호스팅 구성]

my-bucket/
  ├── index.html
  ├── style.css
  ├── app.js
  └── images/
       └── cake.jpg

→ 버킷을 정적 웹사이트 모드로 설정
→ URL: https://[버킷명].kr.object.ncloudstorage.com/index.html
```

- 별도 서버 없이 HTML/CSS/JS 직접 서빙
- CDN과 연동 시 더 빠른 응답

---

## 정리: Object Storage vs Archive Storage 비교

| 항목 | **Object Storage** | **Archive Storage** |
|------|------------------|-------------------|
| **목적** | 범용 오브젝트 스토리지 | 데이터 아카이빙 및 장기 백업 특화 |
| **저장 비용** | 중간 | **저렴** |
| **API 비용** | 중간 | **비쌈** |
| **입출력 속도** | 빠름 | 상대적으로 느림 |
| **최소 보관 기간** | 없음 | **없음** (타 클라우드는 있는 경우도) |
| **정적 웹 호스팅** | 가능 | 불가 |
| **CLI** | **AWS CLI** | **OpenStack swift CLI** |
| **API** | S3 API | S3 API + swift API |
| **대용량 업로드** | 멀티파트 업로드 (최대 10TB) | DLO/SLO (세그먼트 최대 **5GB**, 5GB 초과 시 분할 필수) |
| **Sub Account 연동** | 지원 | 지원 |

### Archive Storage 특징 (보충)

- 콘솔, API(swift, s3), CLI, SDK 모두 이용 가능
- 데이터 최소 보관 기간 없이 사용 가능
- 오브젝트 생명주기 관리 지원
- Sub Account 연동을 통한 권한 관리 기능 제공
- 높은 내구성 + 합리적인 비용
- 페타바이트(PB) 규모까지 확장 가능한 스토리지 구성
- 긴급 다운로드 지원: 아카이빙된 데이터가 긴급하게 필요할 경우 수분 내에 다운로드 가능
- 데이터 전송 시 SSL 지원

### Archive Storage 대용량 파일 업로드: DLO/SLO

대용량 파일은 여러 개의 Segment Object로 분할하여 동시에 업로드할 수 있다.

- 저장 가능한 Segment의 최대 크기: **5GB**
- **5GB를 초과하는 오브젝트는 반드시 분할하여 업로드**해야 한다
- 파일 하나의 크기가 너무 커서 업로드 속도가 느릴 경우, 속도 향상을 위해 **멀티파트 업로드** 기능 제공

> **DLO**: Dynamic Large Object  
> **SLO**: Static Large Object

*(Archive Storage 상세는 Chapter 8에서)*

---

## 시험 대비 체크리스트

- [ ] Object Storage: **AWS S3 호환**, **AWS CLI** 사용
- [ ] 정적 웹 호스팅: **가능**
- [ ] 버킷 최대 개수: 계정당 **1,000개**
- [ ] 객체명 최대 길이: **1,024 Byte**
- [ ] 콘솔 업로드 최대: **1GB**
- [ ] API 업로드 최대: **10TB** (PutObject, UploadPart)
- [ ] 콘솔 다운로드 최대: **2GB**
- [ ] API 다운로드: **제한 없음** (getObject)
- [ ] 버킷 저장 용량: **제한 없음**
- [ ] Linux 마운트: **s3fuse, goofys** 사용 가능 (단, 안정성 보장 없음, 속도 느림)
- [ ] 공개 관리: 버킷 공개(목록 확인 + 업로드) / 파일 공개(열람 + 다운로드)
- [ ] 권한 관리(ACL): 버킷 단위(목록조회, 업로드, ACL조회/수정) / 파일 단위(다운로드, ACL조회/수정)
- [ ] Lifecycle: Object → Archive 자동 이동 가능 (스케줄 기반)
- [ ] Archive Storage: CLI는 **OpenStack swift**, 세그먼트 최대 **5GB**, 5GB 초과 시 분할 필수

---

## 암기 핵심 문장

> "**Object Storage = AWS CLI, 버킷 1,000개, 콘솔 1GB, API 10TB.**"  
> "**정적 웹 호스팅 가능한 스토리지 = Object Storage.**"  
> "**Archive Storage 세그먼트 최대 5GB, 초과 시 DLO/SLO로 분할 업로드.**"

---

*다음 챕터: `08_Archive_NAS.md` → 깊은 저장소와 공유 창고*
