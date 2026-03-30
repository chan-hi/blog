# cloudops blog — 프로젝트 기획서

## 블로그 개요

- **블로그명**: cloudops
- **운영자**: 클라우드 엔지니어
- **목적**: 클라우드 실무 경험 및 기술 지식 공유
- **배포**: GitHub Pages
- **도메인**: `{username}.github.io` 또는 커스텀 도메인

---

## 기술 스택

| 항목 | 선택 | 이유 |
|------|------|------|
| SSG | **Jekyll** (Chirpy 테마) | GitHub Pages 기본 지원, 설정 최소화, 기술 블로그에 최적화 |
| 댓글 | **Giscus** (GitHub Discussions 기반) | 별도 계정 불필요, GitHub 생태계 통합 |
| 검색 | 내장 검색 (Chirpy 기본 제공) | 태그/카테고리 기반 빠른 탐색 |
| 분석 | **Google Analytics 4** | 방문자 통계 |
| CI/CD | **GitHub Actions** | 자동 빌드/배포 |

> 대안: Hugo (속도 빠름) / Astro (모던 JS 기반) — Jekyll이 GitHub Pages 네이티브라 가장 마찰 적음

---

## 카테고리 구조

```
cloudops/
├── AWS              # EC2, S3, VPC, EKS, IAM, Cost Explorer 등
├── NCP              # Naver Cloud Platform 실무
├── NHN Cloud        # NHN Cloud 아키텍처 및 운영
├── Kubernetes       # 클러스터 구성, Helm, Operator, 트러블슈팅
├── IaC              # Terraform, Pulumi, Ansible, CloudFormation
├── DevOps           # CI/CD, GitOps, ArgoCD, Jenkins, GitHub Actions
├── FinOps           # 클라우드 비용 최적화, Reserved Instance, Spot
├── Migration        # 온프레미스 → 클라우드, 멀티클라우드 전략
├── System           # 리눅스, 네트워크, 스토리지, 보안
└── Tips             # 실무 팁, 도구 추천, 트러블슈팅 모음
```

---

## 디렉토리 구조 (Jekyll Chirpy 기준)

```
gitblog/
├── _config.yml          # 블로그 기본 설정 (제목, URL, SNS, 댓글 등)
├── _posts/              # 포스트 (YYYY-MM-DD-title.md)
├── _tabs/               # 사이드바 탭 (about, categories, tags, archives)
├── assets/
│   └── img/
│       ├── posts/       # 포스트 이미지
│       └── avatars/     # 프로필 이미지
├── .github/
│   └── workflows/
│       └── pages-deploy.yml   # 자동 배포 워크플로
└── Gemfile
```

---

## 포스트 작성 규칙

### 파일명
```
YYYY-MM-DD-{slug}.md
예) 2026-03-30-eks-node-group-upgrade.md
```

### Front Matter 템플릿
```yaml
---
title: "포스트 제목"
date: YYYY-MM-DD HH:MM:SS +0900
categories: [상위카테고리, 하위카테고리]   # 최대 2단계
tags: [tag1, tag2, tag3]
description: "SEO용 한 줄 요약"
image:
  path: /assets/img/posts/thumbnail.png
  alt: "이미지 설명"
---
```

### 포스트 품질 기준
- 실무에서 실제로 겪은 문제 또는 검증된 내용만 작성
- 명령어/코드 블록에 언어 지정 필수 (`bash`, `yaml`, `hcl`, `python`)
- 아키텍처 다이어그램은 draw.io 또는 Mermaid 활용
- 비용 관련 수치는 작성 시점 명시

---

## 셋업 순서

### 1단계 — GitHub Repository 생성
```
Repo명: {username}.github.io
Settings → Pages → Source: GitHub Actions
```

### 2단계 — Chirpy Starter 클론
```bash
git clone https://github.com/cotes2020/chirpy-starter.git {username}.github.io
cd {username}.github.io
bundle install
```

### 3단계 — _config.yml 핵심 설정
```yaml
title: cloudops
tagline: "클라우드 엔지니어의 기술 블로그"
description: "AWS, NCP, NHN, Kubernetes, IaC, FinOps 실무 기록"
url: "https://{username}.github.io"
author:
  name: cloudops
timezone: Asia/Seoul
lang: ko-KR
comments:
  provider: giscus
  giscus:
    repo: "{username}/{username}.github.io"
```

### 4단계 — 로컬 미리보기
```bash
bundle exec jekyll serve
# http://localhost:4000 에서 확인
```

### 5단계 — GitHub에 push → 자동 배포
```bash
git add .
git commit -m "init: cloudops blog"
git push origin main
# GitHub Actions가 자동으로 빌드 & 배포
```

---

## 첫 포스트 추천 주제

1. **블로그 개설 소개** — cloudops 블로그를 시작하며
2. **AWS vs NCP vs NHN Cloud 비교** — 국내 엔지니어 관점에서
3. **Terraform으로 EKS 클러스터 구성하기** — 실전 IaC
4. **클라우드 비용 30% 줄인 FinOps 전략** — Reserved + Spot 혼합
5. **온프레미스 → AWS 마이그레이션 체크리스트**

---

## 운영 목표

- 월 2~4회 이상 포스트 업로드
- 각 클라우드 플랫폼(AWS/NCP/NHN) 카테고리 균형 유지
- 실무 코드/설정 파일은 포스트 내 코드블록으로 삽입
- 시리즈 형태로 연재 (예: "Kubernetes 운영 시리즈 1~N편")

---

## 참고

- Chirpy Starter: `https://github.com/cotes2020/chirpy-starter`
- Chirpy 테마 문서: `https://chirpy.cotes.page`
- Giscus 설정: `https://giscus.app`
