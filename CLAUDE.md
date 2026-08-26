# cloudops blog — 프로젝트 기획서 및 현황

## 현재 상태 (2026-03-30 기준)

| 항목 | 상태 | 비고 |
|------|------|------|
| GitHub 레포 | ✅ 완료 | `chan-hi/blog` (Public) |
| Jekyll Chirpy 설치 | ✅ 완료 | chirpy-starter 기반 |
| `_config.yml` 설정 | ✅ 완료 | cloudops 설정 적용 |
| GitHub Pages 활성화 | ✅ 완료 | GitHub Actions 방식 |
| 자동 배포 | ✅ 완료 | push → 자동 빌드/배포 |
| 블로그 URL | ✅ 운영 중 | `https://chan-hi.github.io/blog/` |
| 포스트 폴더 구조 | ✅ 완료 | 카테고리별 폴더 분리 |
| 댓글 (Giscus) | ⬜ 미설정 | 추후 설정 필요 |
| Google Analytics | ⬜ 미설정 | 추후 설정 필요 |
| About 페이지 | ⬜ 미작성 | `_tabs/about.md` 수정 필요 |
| 프로필 이미지 | ⬜ 미설정 | `_config.yml` avatar 항목 |

---

## 블로그 정보

- **블로그명**: cloudops
- **URL**: `https://chan-hi.github.io/blog/`
- **GitHub 레포**: `https://github.com/chan-hi/blog`
- **로컬 경로**: `/home/claude/gitblog/`
- **브랜치**: `main`

---

## 기술 스택

| 항목 | 선택 |
|------|------|
| SSG | Jekyll + Chirpy 테마 |
| 배포 | GitHub Pages + GitHub Actions |
| 댓글 | Giscus (미설정) |
| 분석 | Google Analytics 4 (미설정) |

---

## 레포 디렉토리 구조

```
gitblog/
├── _config.yml               # 블로그 전체 설정
├── _posts/                   # 포스트 (카테고리별 폴더로 분리)
│   ├── aws/
│   ├── ncp/
│   │   ├── 2026-03-30-ncp-dms-migration-guide.md
│   │   └── 2026-03-30-ncp-dms-troubleshooting-summary.md
│   ├── nhn/
│   ├── kubernetes/
│   ├── iac/
│   ├── devops/
│   ├── finops/
│   ├── migration/
│   ├── system/
│   └── tips/
├── _tabs/                    # 사이드바 메뉴 페이지
│   ├── about.md              ← 소개 페이지 (미작성)
│   ├── categories.md
│   ├── tags.md
│   └── archives.md
├── _data/
│   ├── contact.yml           # 연락처 아이콘 설정
│   └── share.yml             # 공유 버튼 설정
├── assets/                   # 이미지, CSS, JS
├── .github/workflows/
│   └── pages-deploy.yml      # 자동 배포 워크플로
└── Gemfile
```

---

## 포스트 작성 방법

### 1. 파일 생성
카테고리 폴더 안에 파일 생성:
```
_posts/{카테고리}/YYYY-MM-DD-{slug}.md
예) _posts/kubernetes/2026-04-01-eks-node-upgrade.md
```

### 2. Front Matter 템플릿
```yaml
---
title: "포스트 제목"
date: YYYY-MM-DD HH:MM:SS +0900
categories: [상위카테고리, 하위카테고리]
tags: [tag1, tag2, tag3]
description: "SEO용 한 줄 요약"
---
```

### 3. Push → 자동 배포
```bash
git add .
git commit -m "post: 포스트 제목"
git push origin main
# 1~3분 후 블로그에 반영
```

---

## 카테고리 구조

| 폴더 | 카테고리 | 주요 주제 |
|------|---------|---------|
| `aws/` | AWS | EC2, S3, VPC, EKS, IAM 등 |
| `ncp/` | NCP | Naver Cloud Platform 실무 |
| `nhn/` | NHN Cloud | NHN Cloud 아키텍처 및 운영 |
| `kubernetes/` | Kubernetes | 클러스터, Helm, Operator |
| `iac/` | IaC | Terraform, Ansible, Pulumi |
| `devops/` | DevOps | CI/CD, GitOps, ArgoCD |
| `finops/` | FinOps | 비용 최적화, Reserved/Spot |
| `migration/` | Migration | 온프레미스 → 클라우드 |
| `system/` | System | 리눅스, 네트워크, 보안 |
| `tips/` | Tips | 실무 팁, 트러블슈팅 |

---

## 업로드된 포스트

| 날짜 | 파일 | 카테고리 |
|------|------|---------|
| 2026-03-30 | ncp-dms-migration-guide | NCP, Migration |
| 2026-03-30 | ncp-dms-troubleshooting-summary | NCP, Migration |

### 실무 경험 시리즈 (2026-08-26)

MSP 클라우드 엔지니어 실무 기록. **고객사명·계약 금액·식별 가능한 인프라 규모는 전부 제외**하고
업종 수준으로만 표기한다. 신규 포스트 작성 시 이 기준을 유지할 것.

| 날짜 | 파일 | 카테고리 |
|------|------|---------|
| 2026-08-26 | system/nat-gateway-nacl-ephemeral-port-mismatch | Troubleshooting, Network |
| 2026-08-26 | iac/terraform-server-scaleout-lb-target-integration | IaC, Terraform |
| 2026-08-26 | finops/rightsizing-min-max-avg-decomposition | FinOps, 비용최적화 |
| 2026-08-26 | migration/dns-migration-700-records-verification | Migration, DNS |
| 2026-08-26 | kubernetes/kubernetes-enablement-instead-of-building | Kubernetes, Enablement |
| 2026-08-26 | migration/idc-to-cloud-db-cutover-decision-criteria | Migration, 아키텍처 |
| 2026-08-26 | ncp/security-required-3tier-design | NCP, 아키텍처 |
| 2026-08-26 | system/postgresql-cpu-90-vacuum-tuning | Troubleshooting, Database |
| 2026-08-26 | system/incident-response-casebook-2026 | Troubleshooting, 운영 |

### 공개 원칙 (실무 포스트)

| 항목 | 처리 |
|------|------|
| 고객사명 | 업종으로만 표기 (금융권, 유통, 미디어, 공공 모빌리티, 제조 SaaS) |
| 재직사명 | 표기하지 않음 |
| 계약·수주 대가 금액 | 제외 (절감 성과액은 업종만 표기된 상태에서 허용) |
| 공수·인력 산정 수치 | 구체값 제외, 방법론만 서술 |
| 인프라 절대 규모 | 근사치로만 ("50여 대", "수 TB") |
| IP·도메인·호스트명 | 마스킹 |
| 확인되지 않은 원인 | 단정하지 않고 미확인으로 명시 |

---

## 남은 작업

- [ ] `_tabs/about.md` — 소개 페이지 작성
- [ ] `_config.yml` — 프로필 이미지(avatar) 추가
- [ ] Giscus 댓글 설정 (`https://giscus.app`)
- [ ] Google Analytics 4 연결

---

## 참고

- Chirpy 문서: `https://chirpy.cotes.page`
- Giscus 설정: `https://giscus.app`
- GitHub Actions 현황: `https://github.com/chan-hi/blog/actions`
