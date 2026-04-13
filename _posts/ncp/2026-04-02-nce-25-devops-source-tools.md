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

## Developer Tools 개요

Developer Tools는 **설계 → 개발 → 테스트 → 생산 및 운영**까지 어플리케이션 라이프사이클 전반에 보안을 통합하여 다음과 같은 효과를 얻을 수 있는 서비스다.

- **보안 리스크 최소화** — 개발 생명주기 전 단계에 보안을 내재화
- **컴플라이언스 비용 절감** — 보안 상품 연계로 자동화된 위협 방지
- **소프트웨어의 빠른 배포** — 반복 프로세스 자동화로 신속한 기능 출시 및 버그 수정
- **반응 및 복구 속도 개선** — 실시간 로그 확인과 즉각적인 대응 체계

### Developer Tools 특장점

**안정성과 유연성을 갖춘 개발 환경**  
네이버의 오랜 경험이 쌓인 노하우가 축적된 DevTools 상품들을 제공한다. 작은 Pilot 개발 환경에서부터 대규모 개발 환경까지 안정적으로 제공하며, 빠르게 변화하는 시장의 요구에 맞춰 고객이 경쟁력 있는 제품을 안정적으로 개발하고 출시할 수 있도록 돕는다.

**높은 보안성을 지닌 서비스 개발 환경**  
서비스 개발부터 출시에 이르는 전 과정에서 언제나 안전한 개발 환경을 지원한다. 네이버 클라우드 플랫폼의 보안 상품(File Safer 등)과 연계하여 보안 위협을 사전에 방지하고 정보를 안전하게 보호한다.

---

## 전체 흐름 먼저 이해하기

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
       │ 빌드 결과물 → Object Storage / Container Registry
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

## 서비스 1: SourceCommit — 프라이빗 Git 리포지토리

### 개요

소스 코드와 다양한 파일들을 안전하게 저장할 수 있는 **완전 관리형 프라이빗 Git 리포지토리** 서비스다. NCP 환경에 특화되어 있으며, 모든 Git 명령어와 Git 클라이언트를 그대로 사용할 수 있다.

### 2가지 접근 방법

```
1. 네이버 클라우드 플랫폼 콘솔
   - 리파지토리 생성 및 삭제
   - 서브 계정 추가 및 권한 변경
   - Git 주소 확인
   - 소스 코드, 커밋 내용, 브랜치 그래프 확인
   - Git Client에서 사용할 Git 계정 설정

2. 개인 PC (Git Client)
   - SourceCommit에서 생성한 리파지토리를 Git Client로 다운로드 및 변경
   - 콘솔 리파지토리 리스트에서 GIT URL 클릭, 또는 코드 페이지에서 CLONE URL 버튼으로 Git 주소 확인
   - Git Client에서 사용하는 계정 = SourceCommit 콘솔에서 설정한 Git 계정
```

### 워크플로우

SourceCommit의 실제 운영 흐름은 다음과 같다.

1. **관리자** (고객 계정 또는 관리자 권한을 가진 서브 계정)가 콘솔에서 리파지토리를 생성하고, 서브 계정에 권한을 할당한다.
2. **권한을 할당받은 서브 계정**이 실제 리파지토리에 데이터를 수정하고 확인하는 작업을 실행한다.

### 사용자별 역할 상세

| 구분 | 콘솔 | 개발자 포털 | 개인 PC (Git Client) |
|------|------|------------|---------------------|
| **고객 계정** (ROLE: 리파지토리 관리자) | 리파지토리 생성/공유/삭제/기본정보 확인 | Admin 권한 | Admin, Write 권한 |
| **서브 계정** (`NCP_SOURCE_COMMIT_MANAGER`, ROLE: 리파지토리 사용자) | Admin: 생성/공유/삭제/기본정보 확인<br>Write/Read: 소스코드·커밋·브랜치 그래프 확인<br>Read: 위 동일 | Admin, Write, Read 권한 | Admin, Write: Clone/Pull/Push<br>Read: Clone/Pull/Fetch |

### 권한 종류 (시험 포인트!)

| 권한 | 설명 |
|------|------|
| **ADMIN** | 읽기+쓰기+설정 변경+삭제. 리파지토리 생성자에게 **자동 부여**. 서브 계정에 권한 부여 가능. Git Client에서 CLONE, PULL, PUSH 모두 사용 가능 |
| **WRITE** | 읽기+쓰기. Git Client에서 CLONE, PULL, PUSH 가능. 콘솔에서 리파지토리 내용 확인 가능 |
| **READ** | 읽기만. Git Client에서 CLONE, PULL만 가능. 콘솔에서 리파지토리 내용 확인 가능 |

### 특장점

**리포지토리 접근 권한 설정**  
Sub Account와 연동하여 서브 계정을 생성하고 사용자별로 리포지토리 접근 권한을 설정 및 제어한다.

**보안성 강화**  
- 범용적인 Git 클라이언트를 통해 SourceCommit에 접근할 때 **클라이언트 전용 비밀번호**를 부여해, 안전하지 않은 클라이언트 사용으로 인한 NCP 계정 유출 사고를 방지한다.
- 로컬 클라이언트와 통신할 때 **HTTPS 프로토콜**을 사용하여 클라우드에 파일을 안전하게 전송한다.

**높은 가용성과 확장성**  
높은 가용성의 서버와 확장성이 뛰어난 **NAS**에 고객의 중요한 소스 코드를 저장한다.

**File Safer 연동**  
- 네이버 클라우드 플랫폼의 악성코드 감염 진단 서비스인 File Safer와 연동하여 안전하게 소스 코드를 관리한다.
- 리포지토리에 새로운 파일이 전송될 때 **클라우드에서 자동으로 악성 여부를 검사**하고, 감염이 탐지되면 관리자에게 메일을 전송한다.

**마크다운 뷰어 제공**  
완벽한 마크다운 뷰어를 제공하여 현재 디렉터리 위치의 README.md 파일을 마크다운 문법이 적용된 화면으로 제공한다.

**커밋 히스토리 및 브랜치 그래프 제공**  
- 네이버 클라우드 플랫폼 웹 콘솔에서 파일 변경 사항, 커밋 히스토리, 브랜치 그래프를 확인할 수 있다.
- 특정 브랜치 혹은 커밋의 파일 리스트, 파일 내용 및 변경 사항 확인 가능하다.
- 상세한 커밋 히스토리 목록과 커밋 메시지 등 다양한 정보를 제공한다.
- 브랜치의 형상을 그래프로 표현하여 다양한 브랜치 상태를 직관적으로 확인할 수 있다.

**대용량 파일: Git-LFS 지원**  
Object Storage와 연동하여 대용량 파일을 Git-LFS(Large File Storage)로 관리할 수 있다.

### 연동 가능 서비스

```
SourceCommit ↔ File Safer        (악성코드 진단)
SourceCommit ↔ Object Storage    (LFS 대용량 파일)
SourceCommit ↔ SourceBuild       (빌드 소스로 사용)
```

---

## 서비스 2: SourceBuild — 자동 빌드

### 개요

독립된 빌드 서버를 실시간으로 생성하여 다수의 빌드 요청을 동시에 처리하는 서비스다. 다양한 언어로 개발된 소스 코드를 쉽고 빠르게 빌드, 테스트, 배포할 수 있다.

> **사전 준비 필수**: SourceBuild를 사용하려면 빌드할 소스 코드가 올라가 있는 **SourceCommit**과, 빌드 결과물을 저장할 **Object Storage**를 먼저 생성해야 한다.

- 빌드 비용: **빌드 스크립트 실행 시작 ~ 결과물 배포 완료까지** (분 단위 과금)

### 빌드 환경 (시험 포인트!)

| 항목 | 내용 |
|------|------|
| **운영 체제** | **Ubuntu 16.04** 만 지원 |
| **빌드 런타임** | base, **java**, **.NET**, android-java, **python**, **nodejs** |

> 빌드 런타임 버전은 최대한 배포 타겟 서버의 버전과 일치하도록 설정한다. (버전 충돌 방지)

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

### 특장점

**빌드 환경 이미지 선택**  
빌드 용량, 소스 코드, 빌드 및 테스트 케이스에 따라 효과적인 빌드 환경 이미지를 선택하여 빌드를 진행한다.

**빌드 결과물 저장**  
- 간단한 설정으로 빌드 결과물을 Object Storage에 배포한다.
- 백업 기능을 사용하면 과거에 배포한 빌드 결과물도 안전하게 보관할 수 있다.

**상세한 빌드 로그 제공**  
- 빌드 환경 콘솔에 노출되는 빌드 로그를 NCP 웹 콘솔에서 **실시간으로 확인** 가능하다.
- 빌드 로그를 확인하여 빌드 오류 등에 빠르게 대응할 수 있다.

**보안성 강화**  
사용자별로 접근 권한이 설정되어 있는 서브 계정을 할당하여 빌드 프로젝트에 접근 통제를 적용한다.

### 권한 관리

| 권한 | 설명 |
|------|------|
| 고객 계정 | 모든 기능 제약 없이 사용 가능 |
| 서브 계정 (전체 권한) | `NCP_INFRA_MANAGER` 권한 필요 → 모든 기능 제약 없이 사용 가능 |
| 서브 계정 (빌드 프로젝트 생성 권한) | `NCP_SOURCE_BUILD_MANGER` 권한 필요 → 자신이 생성한 빌드 프로젝트 관리 및 다른 서브 계정에 공유 가능 |

### 연동 가능 서비스

- **Container Registry** — 컨테이너 이미지 저장
- **Object Storage** — 빌드 결과물 저장
- **Cloud Log Analytics** — 빌드 로그 저장
- **File Safer** — 빌드 파일 악성코드 검사

---

## 서비스 3: SourceDeploy — 자동 배포

### 개요

서버 그룹에 대한 배포를 자동화해주는 배포 서비스다. 미리 설정된 사용자 기반 명령어들을 통해 소스 배포, 실행 및 검증을 자동화하고, 배포 중 서비스 중단 시간을 최소화한다.

> **필수 조건**: 배포하고자 하는 타겟 서버에 **SourceDeploy용 Agent가 반드시 설치**되어야 한다.

### 지원 OS (시험 포인트!)

```
CentOS, Ubuntu, Rocky 8 버전 이하, RHEL(Red Hat Enterprise Linux)
```

> Agent 설치 제약사항에서의 지원 OS: CentOS, Ubuntu

### 지원 배포 환경 (시험 포인트!)

| 배포 타겟 | 지원 여부 |
|---------|----------|
| NCP Server | O |
| Auto Scaling | O |
| Kubernetes Service | O |
| Object Storage | O |
| 외부 서버 | X |

### 구조 계층

```
프로젝트 (Project)
  └─ 스테이지 (Stage)        ← 환경별 구분 (dev/staging/prod)
       └─ 시나리오 (Scenario)  ← 실제 배포 프로세스 정의
```

하나의 프로젝트 안에 다양한 스테이지를 구성하고, 스테이지별로 서버 그룹을 설정한다. 각 스테이지에서 실행할 시나리오를 여러 개 생성하여 다양한 배포 프로세스를 구성 및 실행할 수 있다.

### 배포 소스

```
배포 파일 소스:
  ├─ Object Storage (압축 파일 → 배포 시 자동 다운로드)
  └─ SourceBuild   (가장 최근 성공한 빌드 결과물 자동 조회 후 배포)
```

### 배포 설정 (배포 타겟 × 배포 전략)

| 배포 타겟 | 지원 배포 전략 |
|---------|--------------|
| **Server** | 순차 배포, 동시 배포 |
| **Auto Scaling** | 순차 배포, 동시 배포 |
| **Kubernetes Service** | Rolling Update, Blue/Green, Canary |
| **Object Storage** | (파일 배포) |

### 무중단 배포 방식

| 방식 | 설명 |
|------|------|
| **Rolling Update** | 인스턴스를 순차적으로 교체. v1을 하나씩 종료하며 v2로 전환 |
| **Blue/Green** | v1(Blue) 환경을 유지한 채 v2(Green)를 배포하고, 트래픽을 한 번에 전환. v1은 대기 상태로 롤백 가능 |
| **Canary** | 트래픽 일부(예: 25%)만 v2로 먼저 전환하여 안정성을 확인한 뒤 점진적으로 100%까지 전환 |

### 특장점

**프로젝트별 스테이지 및 시나리오 구성**  
하나의 프로젝트 안에 다양한 스테이지를 구성하고, 각 스테이지에서 실행할 시나리오를 여러 개 생성하여 다양한 배포 프로세스를 실행할 수 있다.

**Object Storage 및 SourceBuild를 통한 배포 파일 선택**  
- Object Storage에 압축 형태로 소스를 업로드해두면 배포 시 자동으로 해당 파일을 다운로드받아 배포한다.
- SourceBuild를 선택한 경우 빌드 프로젝트의 가장 마지막으로 성공한 결과물을 자동으로 배포한다.

**상세 로그 정보 제공**  
배포 타겟 서버에서 발생하는 로그들을 콘솔에서 **실시간으로 확인**하여, 배포 과정에서 발생하는 로그들에 빠르게 대응할 수 있다.

**배포 실행 관리자 설정 가능 (승인 기반 배포)**  
- 배포 실행 관리자를 설정할 수 있어, 특정 스테이지에 대해 발생하는 배포는 **승인을 통해서만 배포가 실행**되도록 제어한다.
- 관리자를 **여러 명** 설정한 경우에는 승인 규칙에 따라 배포를 실행한다.

**사용자별 접근 제어**  
Sub Account와 연동하여 배포 프로젝트별로 접근을 통제한다.

### 권한 관리

| 권한 | 설명 |
|------|------|
| 고객 계정 | 기본 권한이 부여되어 있음 |
| 서브 계정 | `NCP_SOURCE_DEPLOY_MANGER` 권한 필요 |

---

## 서비스 4: SourcePipeline — CI/CD 파이프라인 자동화

### 개요

빠르고 안정적인 소프트웨어 출시를 위한 프로세스 자동화 서비스다. 손쉽게 새로운 기능을 출시할 수 있도록 리파지토리, 빌드, 배포 프로세스를 통합하여 제공한다.

> **사용 조건**: SourcePipeline을 사용하기 위해서는 **SourceCommit, SourceBuild, SourceDeploy** 상품을 모두 이용해야 한다.

### 파이프라인 구성 3단계 (시험 포인트!)

```
[Source Phase]  ← SourceCommit 연동
      ↓
[Build Phase]   ← SourceBuild 연동
      ↓
[Deploy Phase]  ← SourceDeploy 연동
```

각 Phase는 각각 SourceCommit, SourceBuild, SourceDeploy 서비스와 연동된다.

### 주요 기능

| 기능 | 설명 |
|------|------|
| **소스 변경 자동 감지** | SourceCommit에 코드가 Push되면 파이프라인이 자동으로 트리거 |
| **병렬 실행** | 파이프라인 내 배포 작업을 병렬로 실행 가능 |
| **독립적인 설정** | SourceBuild/SourceDeploy 원본 설정 변경 없이 파이프라인에서 별도 설정 적용 가능 |
| **배포 환경 선택** | 스테이지와 시나리오를 구성하여 배포 실행 조절 |

---

## CI/CD 운영에 필요한 기본 Git 명령어

SourceCommit과 SourcePipeline을 실제로 운영하려면 다음 Git 명령어들을 숙지해야 한다.

| 명령어 | 설명 |
|--------|------|
| `git init` | 로컬 서버에 git 리파지토리 생성/초기화 |
| `git add <파일/디렉토리명>` | git 리파지토리에 특정 파일 또는 디렉토리를 스테이징 단계에 추가 |
| `git status` | 스테이징 단계에서의 현재 변경 상태 확인 |
| `git commit` | 리파지토리에 변경된 코드 내용을 저장 (`-m` 옵션: 커밋 메시지 추가, `-a` 옵션: 트래킹하는 모든 파일의 변경사항을 스테이징함과 동시에 커밋) |
| `git log` | 스테이징을 거쳐 커밋된 결과의 히스토리 확인 |
| `git branch` | 브랜치 목록 확인 |
| `git branch <브랜치명>` | 신규 브랜치 생성 |
| `git checkout <브랜치명>` | 브랜치 간 이동 |
| `git rm <파일/디렉토리명>` | 파일 또는 디렉터리를 삭제한 내역을 스테이징 단계에 반영 |
| `git push <원격 리파지토리명> <브랜치명>` | 커밋한 결과를 원격 리파지토리의 특정 브랜치로 전송 |
| `git pull <원격 리파지토리명> <브랜치명>` | 원격 리파지토리의 특정 브랜치 내용을 가져옴 |

---

## Source 4총사 핵심 비교

| 서비스 | 역할 | 주요 제약 |
|--------|------|----------|
| **SourceCommit** | 코드 저장 (Git) | 권한 3종: ADMIN/WRITE/READ, HTTPS 통신 |
| **SourceBuild** | 자동 빌드 | OS: Ubuntu 16.04만 지원, GitHub도 소스 가능, 사전에 SourceCommit+Object Storage 필요 |
| **SourceDeploy** | 자동 배포 | OS: CentOS/Ubuntu/Rocky8이하/RHEL, 4가지 타겟 지원, Agent 설치 필수 |
| **SourcePipeline** | 파이프라인 통합 | 3개 서비스 모두 필요, 병렬 실행 지원 |

---

## 실무 시나리오: 전체 파이프라인 구성

```
1. SourceCommit에 코드 저장
   - 개발팀이 기능 개발 후 git push
   - ADMIN이 코드 리뷰 후 Merge
   - File Safer가 자동으로 악성코드 검사 후 감염 시 관리자에게 메일 발송

2. SourceBuild로 자동 빌드
   - Ubuntu 16.04 환경에서 빌드 (runtime: java/python/nodejs 등 선택)
   - 빌드 로그 웹 콘솔에서 실시간 확인
   - 빌드 결과물 → Object Storage 저장 (백업 포함)

3. SourceDeploy로 자동 배포
   - staging 스테이지 → prod 스테이지 순서로 배포
   - prod 스테이지에 배포 실행 관리자 설정 → 승인 후 배포 실행
   - Kubernetes 환경이라면 Blue/Green 또는 Canary 배포 전략 선택

4. SourcePipeline으로 통합 자동화
   - 코드 Push 감지 → Source Phase → Build Phase → Deploy Phase 자동 실행
   - 오류 발생 시 즉각 통보
   - 여러 배포 작업은 병렬로 실행 가능
```

---

## 시험 직전 체크리스트

- [ ] SourceBuild OS: Ubuntu **16.04**만 지원
- [ ] SourceBuild 빌드 소스: **SourceCommit** 외에 **GitHub**도 가능
- [ ] SourceBuild 빌드 전 사전 생성 필요: **SourceCommit + Object Storage**
- [ ] SourceBuild 빌드 환경 이미지: SourceCommit 관리 이미지, Container Registry, Public Registry 3가지
- [ ] SourceDeploy 지원 OS: **CentOS, Ubuntu, Rocky 8 이하, RHEL**
- [ ] SourceDeploy Agent 설치 지원 OS: **CentOS, Ubuntu**
- [ ] SourceDeploy 배포 타겟 4가지: Server, Auto Scaling, Kubernetes Service, Object Storage
- [ ] SourceDeploy 무중단 배포 3가지: **Rolling Update, Blue/Green, Canary** (Kubernetes에서 사용)
- [ ] SourceDeploy 배포 실행 관리자: 여러 명 설정 가능, 승인 규칙에 따라 실행
- [ ] SourceDeploy 타겟 서버에 **Agent 설치 필수**
- [ ] SourceCommit 권한 3종: ADMIN(생성자 자동 부여), WRITE, READ
- [ ] SourceCommit 보안: **HTTPS 통신**, 클라이언트 전용 비밀번호 부여
- [ ] SourceCommit File Safer 연동: 새 파일 전송 시 **자동 악성코드 검사**, 감염 시 관리자 메일 발송
- [ ] SourcePipeline 구성 3단계: Source → Build → Deploy Phase
- [ ] SourcePipeline 사용 조건: **3개 서비스 모두** 이용해야 함
- [ ] SourceBuild 권한: 서브계정 전체 권한 → `NCP_INFRA_MANAGER`
- [ ] SourceBuild 권한: 서브계정 프로젝트 생성 → `NCP_SOURCE_BUILD_MANGER`
- [ ] SourceDeploy 권한: 서브계정 → `NCP_SOURCE_DEPLOY_MANGER`
- [ ] SourceCommit 서브계정 권한: `NCP_SOURCE_COMMIT_MANAGER`
- [ ] SourcePipeline: 병렬 실행 지원

---

**참고 문서**:
- [SourceCommit 개요](https://guide.ncloud-docs.com/docs/sourcecommit-overview)
- [SourceBuild 개요](https://guide.ncloud-docs.com/docs/sourcebuild-overview)
- [SourceDeploy 개요](https://guide.ncloud-docs.com/docs/sourcedeploy-overview)
- [SourcePipeline 개요](https://guide.ncloud-docs.com/docs/sourcepipeline-overview)
