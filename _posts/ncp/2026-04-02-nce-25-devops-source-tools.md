---
title: "Chapter 25: 코드에서 배포까지 — NCP DevOps Source 4총사"
date: 2026-04-02 10:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, devops, sourcecommit, sourcebuild, sourcedeploy, sourcepipeline, cicd]
description: "NCP NCE 자격증 대비 학습 자료 - SourceCommit, SourceBuild, SourceDeploy, SourcePipeline CI/CD 파이프라인 완전정복"
---
> **학습 목표**: NCP DevTools 4종(SourceCommit → SourceBuild → SourceDeploy → SourcePipeline)의 역할과 특징, 제약사항을 파이프라인 흐름에서 이해한다.

---

## 이야기: "클라우드빵집의 코드 배포 대참사"

클라우드빵집 개발팀은 매번 배포할 때마다 고통받았다.

> "지난번에 서버에 직접 SSH로 올렸다가 운영 DB가 날아갔잖아요..."  
> "누가 어디에 어떤 코드 올렸는지도 몰라."  
> "테스트도 없이 그냥 배포하면 어떡해!"

팀 리더가 NCP Developer Tools를 켰다.

```
SourceCommit   ← 코드 저장소 (Git)
SourceBuild    ← 자동 빌드
SourceDeploy   ← 자동 배포
SourcePipeline ← 위 세 개를 하나의 파이프라인으로
```

> "이제 코드 한 줄 올리면 자동으로 빌드하고 배포까지 된다."

---

## 🔑 전체 흐름 먼저 이해하기

```
[개발자 PC]
     │  git push
     ▼
┌─────────────┐
│ SourceCommit│  ← 소스 코드 저장 (프라이빗 Git 리포지토리)
└──────┬──────┘
       │ 소스 전달
       ▼
┌─────────────┐
│ SourceBuild │  ← 빌드 실행 (컴파일, 테스트, 패키징)
└──────┬──────┘
       │ 빌드 결과물
       ▼
┌──────────────┐
│ SourceDeploy │  ← 서버에 자동 배포
└──────┬───────┘
       │
       ▼
   [운영 서버]

위 전체를 SourcePipeline이 한 번에 자동화!
```

---

## 🔑 서비스 1: SourceCommit — 프라이빗 Git 리포지토리

### 특징

- NCP 환경에 특화된 **완전 관리형 프라이빗 Git 리포지토리** 서비스
- 모든 Git 명령어 지원 + 모든 Git 클라이언트 호환
- **File Safer** 연동 → 커밋 단위 악성코드 진단
- 대용량 파일: **Git-LFS** 지원 (Object Storage 연동)

### 2가지 접근 방법

```
1. 네이버 클라우드 플랫폼 콘솔
   - 리파지토리 생성/삭제
   - 서브 계정 추가 및 권한 변경
   - 소스 코드, 커밋 내용, 브랜치 그래프 확인
   - Git 계정 설정

2. 개인 PC (Git Client)
   - CLONE, PULL, PUSH 등 표준 Git 명령어 사용
   - Git Client에서 사용하는 계정 = SourceCommit 콘솔에서 설정한 Git 계정
```

### 권한 종류 (시험 포인트!)

| 권한 | 설명 |
|------|------|
| **ADMIN** | 읽기+쓰기+설정 변경+삭제. 리파지토리 생성자에게 **자동 부여**. 서브 계정에 권한 부여 가능 |
| **WRITE** | 읽기+쓰기. CLONE, PULL, PUSH 가능 |
| **READ** | 읽기만. CLONE, PULL만 가능 |

### 연동 가능 서비스

```
SourceCommit ↔ File Safer     (악성코드 진단)
SourceCommit ↔ Object Storage (LFS 대용량 파일)
SourceCommit ↔ SourceBuild    (빌드 소스로 사용)
```

---

## 🔑 서비스 2: SourceBuild — 자동 빌드

### 특징

- 독립된 빌드 서버를 **실시간으로 생성**, 다수 빌드 동시 처리
- 빌드 비용: **빌드 스크립트 실행 시작 ~ 결과물 배포 완료까지** (분 단위)

### 빌드 환경 (시험 포인트!)

| 항목 | 내용 |
|------|------|
| **운영 체제** | **Ubuntu 16.04** 만 지원 |
| **빌드 런타임** | base, **java**, **.NET**, android-java, **python**, **nodejs** |

### 빌드 소스 종류 (시험 포인트!)

```
빌드 대상 소스:
  ├─ SourceCommit (NCP 내부)
  └─ GitHub (외부도 가능!)

빌드 환경 이미지:
  ├─ SourceCommit에서 관리되는 이미지
  ├─ Container Registry 이미지
  └─ Public Registry 이미지
```

### 연동 가능 서비스

- **Container Registry** — 컨테이너 이미지 저장
- **Object Storage** — 빌드 결과물 저장
- **Cloud Log Analytics** — 빌드 로그 저장
- **File Safer** — 빌드 파일 악성코드 검사

### 권한 관리

| 권한 | 설명 |
|------|------|
| 고객 계정 | 모든 기능 제약 없이 사용 가능 |
| 서브 계정 (전체 권한) | `NCP_INFRA_MANAGER` 권한 필요 |
| 서브 계정 (프로젝트 생성만) | `NCP_SOURCE_BUILD_MANGER` 권한 필요 |

---

## 🔑 서비스 3: SourceDeploy — 자동 배포

### 특징

- 빌드 결과물을 자동으로 서버에 **배포**하는 서비스
- 배포 중 **서비스 중단 시간 최소화**
- 배포 실행 관리자를 통한 **승인 기반 배포** 지원

### 구조 계층

```
프로젝트 (Project)
  └─ 스테이지 (Stage)        ← 환경별 구분 (dev/staging/prod)
       └─ 시나리오 (Scenario)  ← 실제 배포 프로세스 정의
```

### 배포 소스

```
배포 파일 소스:
  ├─ Object Storage (압축 파일 자동 다운로드)
  └─ SourceBuild (가장 최근 성공한 빌드 결과물)
```

### 지원 배포 환경 (시험 포인트!)

| 배포 타겟 | 지원 여부 |
|---------|----------|
| NCP Server | **O** |
| Auto Scaling | **O** |
| Kubernetes Service | **O** |
| Object Storage | **O** |
| 외부 서버 | X |

### 지원 OS (시험 포인트!)

```
CentOS, Ubuntu
```

### 권한 관리

| 권한 | 설명 |
|------|------|
| 고객 계정 | 기본 권한 부여됨 |
| 서브 계정 | `NCP_SOURCE_DEPLOY_MANGER` 권한 필요 |

---

## 🔑 서비스 4: SourcePipeline — CI/CD 파이프라인 자동화

### 특징

- **SourceCommit + SourceBuild + SourceDeploy**를 통합한 자동화 서비스
- 소스 변경 감지 → 빌드 → 배포까지 **완전 자동화**

### 파이프라인 구성 3단계 (시험 포인트!)

```
[Source Phase]  ← SourceCommit 연동
      ↓
[Build Phase]   ← SourceBuild 연동
      ↓
[Deploy Phase]  ← SourceDeploy 연동
```

### 주요 기능

| 기능 | 설명 |
|------|------|
| **병렬 실행** | 파이프라인 내 배포 작업을 병렬로 실행 가능 |
| **독립적인 설정** | SourceBuild/SourceDeploy 원본 설정 변경 없이 파이프라인에서 별도 설정 적용 가능 |
| **배포 환경 선택** | 스테이지와 시나리오를 구성하여 배포 실행 조절 |

> 💡 **사용 조건**: SourcePipeline을 쓰려면 반드시 **SourceCommit, SourceBuild, SourceDeploy** 모두 이용해야 함!

---

## 🧠 Source 4총사 핵심 비교

| 서비스 | 역할 | 주요 제약 |
|--------|------|----------|
| **SourceCommit** | 코드 저장 (Git) | 권한 3종: ADMIN/WRITE/READ |
| **SourceBuild** | 자동 빌드 | OS: Ubuntu 16.04만, GitHub도 소스 가능 |
| **SourceDeploy** | 자동 배포 | OS: CentOS/Ubuntu, 4가지 타겟 지원 |
| **SourcePipeline** | 파이프라인 통합 | 3개 서비스 모두 필요, 병렬 실행 지원 |

---

## 실무 시나리오: 전체 파이프라인 구성

```
1. SourceCommit에 코드 저장
   - 개발팀이 기능 개발 후 Push
   - ADMIN이 코드 리뷰 후 Merge

2. SourceBuild로 자동 빌드
   - Ubuntu 16.04 환경에서 빌드
   - 빌드 결과물 → Object Storage 저장

3. SourceDeploy로 자동 배포
   - staging 스테이지 → prod 스테이지 순서로 배포
   - 관리자 승인 후 prod 배포 실행

4. SourcePipeline으로 통합 자동화
   - 코드 Push 감지 → 빌드 → 배포 자동 실행
   - 오류 발생 시 즉각 통보
```

---

## 시험 직전 체크리스트

- [ ] SourceBuild OS: Ubuntu **16.04**만 지원
- [ ] SourceBuild 빌드 소스: **SourceCommit** 외에 **GitHub**도 가능
- [ ] SourceDeploy 지원 OS: **CentOS, Ubuntu**
- [ ] SourceDeploy 배포 타겟 4가지: Server, Auto Scaling, Kubernetes, Object Storage
- [ ] SourceCommit 권한 3종: ADMIN(생성자 자동), WRITE, READ
- [ ] SourcePipeline 구성 3단계: Source → Build → Deploy Phase
- [ ] SourcePipeline 사용 조건: **3개 서비스 모두** 이용해야 함
- [ ] SourceBuild 권한: 서브계정 전체 권한 → `NCP_INFRA_MANAGER`
- [ ] SourceDeploy 권한: 서브계정 → `NCP_SOURCE_DEPLOY_MANGER`
- [ ] SourcePipeline: 병렬 실행 지원

---

**참고 문서**:
- [SourceCommit 개요](https://guide.ncloud-docs.com/docs/sourcecommit-overview)
- [SourceBuild 개요](https://guide.ncloud-docs.com/docs/sourcebuild-overview)
- [SourceDeploy 개요](https://guide.ncloud-docs.com/docs/sourcedeploy-overview)
- [SourcePipeline 개요](https://guide.ncloud-docs.com/docs/sourcepipeline-overview)
