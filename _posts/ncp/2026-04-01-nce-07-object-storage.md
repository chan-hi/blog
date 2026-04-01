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

---

## 🔑 핵심 개념 1: Object Storage 기본 특징

| 항목 | 내용 |
|------|------|
| **API 호환** | **AWS S3 API** 호환 |
| **CLI 도구** | **AWS CLI** (NCP 자체 CLI 아님!) |
| **Lifecycle** | 데이터 자동 이동/삭제 주기 설정 가능 |
| **정적 웹 호스팅** | ✅ 가능 (HTML, CSS, JS, 이미지 직접 서빙) |
| **데이터 복구** | ❌ 삭제 시 복구 불가 (정기 백업 권장) |

> ⚠️ **시험 포인트**  
> Object Storage CLI는 **AWS CLI**다! (S3 호환이기 때문)  
> `aws s3 cp`, `aws s3 ls` 등의 명령어 사용

```bash
# AWS CLI로 Object Storage 사용 예시
aws s3 cp ./photo.jpg s3://my-bucket/ \
    --endpoint-url=https://kr.object.ncloudstorage.com

aws s3 ls s3://my-bucket/ \
    --endpoint-url=https://kr.object.ncloudstorage.com
```

---

## 🔑 핵심 개념 2: 주요 제한 사항 (⭐⭐ 필수 암기!)

숫자를 정확히 외워야 한다. 시험에 거의 매번 출제된다.

| 항목 | **제한 사양** |
|------|-------------|
| 생성 가능한 **버킷 개수** | 계정당 **1,000개** 이하 |
| **객체명 길이** (폴더 경로 포함) | **1,024 Byte** 이하 |
| **콘솔 업로드** 최대 크기 | 단일 파일 **1GB** 이하 |
| **API 업로드** 최대 크기 | 단일 파일 **10TB** 이하 (멀티파트 활용) |
| **콘솔 다운로드** 최대 크기 | **2GB** 이하 |
| **API 다운로드** 최대 크기 | **제한 없음** |
| **버킷 저장 용량** | **제한 없음** |

> 📌 **시험 단골 3문제**  
> Q1: "콘솔로 올릴 수 있는 최대 파일 크기?" → **1GB**  
> Q2: "API로 올릴 수 있는 최대 파일 크기?" → **10TB**  
> Q3: "버킷은 계정당 최대 몇 개?" → **1,000개**

---

## 🔑 핵심 개념 3: 권한 관리

Object Storage의 접근 제어는 두 가지 방식이 있다.

| 구분 | 대상 | 방법 |
|------|------|------|
| **공개 관리 (Public)** | 인터넷 전체 | URL로 누구나 접근 가능하게 공개 |
| **권한 관리 (ACL)** | NCP 내부 사용자 | Sub Account 등에게 세밀한 권한 부여 |

```
[공개 설정 예시]
버킷 전체 공개 → 누구나 URL로 이미지 다운로드 가능
특정 폴더만 공개 → 해당 폴더의 파일만 공개

[ACL 권한 관리]
Sub Account A → my-bucket 읽기 전용
Sub Account B → my-bucket 읽기/쓰기
다른 계정 → 접근 불가
```

---

## 🔑 핵심 개념 4: Lifecycle (수명 주기 관리)

데이터를 자동으로 관리하는 강력한 기능.

```
[Lifecycle 활용 시나리오]

Object Storage에 파일 업로드
       ↓
30일 경과 → Archive Storage로 자동 이동
       ↓
1년 경과 → 자동 삭제

→ 비용 최적화 자동화!
```

> 💡 자주 쓰는 파일은 Object Storage에,  
> 오래된 파일은 Lifecycle으로 자동으로 Archive Storage로 이동시켜 비용 절감.

---

## 🔑 핵심 개념 5: 정적 웹사이트 호스팅

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

## 정리: Object Storage vs Archive Storage 비교 (미리보기)

| 항목 | **Object Storage** | **Archive Storage** |
|------|------------------|-------------------|
| **저장 비용** | 중간 | **저렴** |
| **API 비용** | 중간 | **비쌈** |
| **최소 보관 기간** | 없음 | **없음** (타 클라우드는 있는 경우도) |
| **정적 웹 호스팅** | ✅ 가능 | ❌ 불가 |
| **CLI** | **AWS CLI** | **OpenStack swift** |
| **API** | S3 API | S3 API + swift API |

*(Archive Storage 상세는 Chapter 8에서)*

---

## 📝 시험 대비 체크리스트

- [ ] Object Storage: **AWS S3 호환**, **AWS CLI** 사용
- [ ] 정적 웹 호스팅: **가능** ✅
- [ ] 버킷 최대 개수: 계정당 **1,000개**
- [ ] 객체명 최대 길이: **1,024 Byte**
- [ ] 콘솔 업로드 최대: **1GB**
- [ ] API 업로드 최대: **10TB**
- [ ] 콘솔 다운로드 최대: **2GB**
- [ ] API 다운로드: **제한 없음**
- [ ] 버킷 저장 용량: **제한 없음**
- [ ] Lifecycle: Object → Archive 자동 이동 가능

---

## 🧠 암기 핵심 문장

> "**Object Storage = AWS CLI, 버킷 1,000개, 콘솔 1GB, API 10TB.**"  
> "**정적 웹 호스팅 가능한 스토리지 = Object Storage.**"

---

*다음 챕터: `08_Archive_NAS.md` → 깊은 저장소와 공유 창고*
