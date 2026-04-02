---
title: "Chapter 24: 네이버의 두뇌 — CLOVA AI 서비스 완전정복"
date: 2026-04-02 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, clova, ai, chatbot, ocr, speech, voice, papago, search-trend]
description: "NCP NCE 자격증 대비 학습 자료 - CLOVA Chatbot, OCR, Speech, Voice, Papago, Search Trend 핵심 정리"
---
> **학습 목표**: NCP의 AI 서비스 라인업을 이해하고, 각 서비스의 특징·API 구조·제약사항을 시험에 나오는 포인트 위주로 암기한다.

---

## 이야기: "AI로 만드는 클라우드빵집 서비스"

클라우드빵집이 이번엔 AI를 도입하기로 했다.

> "고객 문의가 너무 많아. 챗봇 만들자."  
> "영수증 자동 인식도 되면 좋겠는데."  
> "외국 고객용 다국어 번역도 필요해."

개발팀이 NCP AI 서비스 목록을 펼쳤다.

```
CLOVA Chatbot  → 대화형 AI 상담봇
CLOVA OCR      → 영수증·문서 텍스트 추출
CLOVA Speech   → 음성 → 텍스트 변환 (STT)
CLOVA Voice    → 텍스트 → 음성 변환 (TTS)
Papago         → 다국어 번역 API
Search Trend   → 네이버 검색 트렌드 분석
```

"네이버가 다 만들어뒀네." 팀 리더가 웃었다.

---

## 🔑 서비스 1: CLOVA Chatbot — 대화형 AI 상담봇

### 핵심 개념

| 항목 | 내용 |
|------|------|
| **정의** | 사용자 질문의 의도를 파악해 적절한 답변을 제공하는 대화형 AI 서비스 |
| **기반 기술** | 네이버 NLU(자연어처리) 엔진 + 딥러닝 앙상블 모델 |
| **지원 언어** | 한국어, 일본어, 영어, 중국어, 태국어, 인도네시아어 (총 **6개 언어**) |

### 자연어 처리 과정 (시험 포인트!)

```
[사용자 질문 입력]
       ↓
[Feature 추출] — 품사, 엔티티, 어미, 형태소 정보
       ↓
[여러 모델에 Feature 전달]
  ┌─ 모델A: 품사 중심 분석
  ├─ 모델B: 엔티티 중심 분석
  └─ 모델C: 기타 분석
       ↓
[앙상블 스코어 계산]
       ↓
[가장 정확한 답변 제공]
```

### 대화 구성 요소

```
도메인(Domain)
  └─ 대화(Dialog)
       ├─ 일반 대화: 발화 → 의도 파악 → 답변 (간단한 Q&A)
       └─ 태스크(Task): 슬롯 수집 → 복잡한 플로우 처리
```

**대화 vs 태스크 선택 기준**:
- **일반 대화**: 단순 Q&A, 대부분의 챗봇이 이것만으로 구성 가능
- **태스크**: 예약·주문처럼 여러 정보(슬롯)를 수집해 특정 작업을 수행할 때

### 챗봇 고급 설정 (시험 포인트!)

| 기능 | 설명 |
|------|------|
| **컨텍스트** | A 대화의 output 컨텍스트 = B 대화의 input 컨텍스트 → B 우선 매칭 |
| **네거티브** | 유사 대화 간 차이를 모델에 학습시킴. 최대 **20개** 등록 |
| **웰컴 메시지** | 세션 시작 시 챗봇이 먼저 건네는 첫 메시지 |
| **실패 메시지** | 답변을 찾지 못했을 때 응답하는 메시지 |
| **무응답 메시지** | AiCall 도메인 전용. 음성 대기 시간 동안 발화 없을 때 |
| **태스크 종료 키워드** | 진행 중인 태스크를 강제로 중단하는 키워드 |
| **유사 답변** | 정확히 매칭 실패 시 '정확도 중간' 구간 대화를 캐로셀로 표시 |
| **피드백** | 사용자가 챗봇 답변을 평가하는 기능 |

### 우선순위 규칙 (반드시 암기!)

> **객관식 버튼 연결 대화** > **컨텍스트 일치 대화** > **일반 대화**

---

## 🔑 서비스 2: CLOVA OCR — 문서/이미지 텍스트 추출

### 서비스 모델 비교 (시험 단골!)

| 항목 | **기본 (Basic)** | **프리미엄 (Premium)** |
|------|-----------------|----------------------|
| **인식 대상** | 활자체만 | 활자체 + **필기체** |
| **적합 문서** | 증명서, 고정 폼 양식 | 수기 신청서, 금융 문서 |
| **멀티박스** | 제공 안함 | **제공** |
| **체크박스** | 제공 안함 | **제공** |
| **필드 유형** | 제공 안함 | **제공** (숫자 전용 등) |
| **인식 템플릿 레이아웃** | 제공 | 제공 |

### API 호출 방식

```
Method: POST
Request URI: CLOVA OCR 빌더에서 생성된 API Gateway InvokeURL
             (도메인마다 고유 URL 생성)
```

### TEXT OCR vs Template OCR

| 구분 | **TEXT OCR** | **Template OCR** |
|------|-------------|-----------------|
| **경로** | `/general` | `/infer` |
| **용도** | 이미지 내 모든 텍스트 인식 | 배포된 템플릿 기반으로 지정 영역 인식 |
| **언어 설정** | `ko` 또는 `ja` (미설정 시 `ko` 디폴트) | 도메인 언어 설정값이 디폴트 |
| **matchedTemplate** | 반환 안됨 | 반환됨 |

> 💡 **ICDAR2019** 4개 분야 1위 기술력 보유

---

## 🔑 서비스 3: CLOVA Speech — 음성 인식 (STT)

### 특징

- **STT(Speech-to-Text)**: 음성 파일/실시간 음성 → 텍스트 변환
- **웹 빌더 제공**: Object Storage에서 파일을 가져오거나 직접 업로드

### API 호출 3가지 방식

```
1. Object Storage 파일 업로드 방식
   POST /recognizer/object-storage

2. 외부 URL 인식 방식
   POST /recognizer/url

3. 로컬 파일 업로드 방식
   POST /recognizer/upload
```

### 주요 파라미터 (시험 포인트!)

| 파라미터 | 설명 | 기본값 |
|---------|------|--------|
| `language` | 인식 언어 | `ko-KR` |
| `completion` | sync/async 방식 | `async` |
| `sttEnable` | 음성 인식 사용 여부 | `true` |
| `diarization.enable` | 화자 인식 여부 | `false` |
| `keywordExtraction.enable` | 키워드 추출 여부 | `false` |

### 키워드 부스팅 제약사항 (시험 포인트!)

- 최대 **500건** 부스팅 가능
- 지원 문자: **한글, 영어, 숫자**만 가능
- `네`, `응`, `no` 같은 **1음절 단어는 부스팅 불가** (오인식 위험)
- 띄어쓰기 여부와 관계없이 부스팅 처리

---

## 🔑 서비스 4: CLOVA Voice — 음성 합성 (TTS)

### 특징

| 항목 | 내용 |
|------|------|
| **정의** | 텍스트 → 자연스러운 음성으로 변환하는 TTS 서비스 |
| **기술** | HDTS, NES(Natural End-to-End Speech Synthesis) |
| **음성 옵션** | **100가지** (성별, 연령대 포괄) |
| **지원 언어** | 한국어, 영어, 일본어, 중국어, 스페인어 등 다국어 |
| **지원 리전** | 한국, 미국, 싱가포르, 일본, 독일 |

### 서비스 플랜 (시험 포인트!)

| 구분 | **Standard** | **Premium** |
|------|-------------|-------------|
| **파일 다운로드** | O | O |
| **실시간 API 호출** | X | X |
| **재가공/재편집** | X | X |
| **개인/비영리 용도** | O | O |
| **키오스크/앱/웹 서비스** | X | △ (제휴 제안 필요) |
| **유료 판매** | X | △ |

> 💡 두 플랜 모두 **실시간 API 호출 불가**, **재가공 불가**

---

## 🔑 서비스 5: Papago — 다국어 번역 API

### 주요 기능

| 기능 | 설명 |
|------|------|
| **번역 API** (NMT API) | 텍스트를 다른 언어로 번역 |
| **언어 감지 API** | 입력 텍스트의 언어를 자동 감지. **12개 언어** 지원 |
| **높임말 번역** | 한국어 번역 시 반말/높임말 옵션 선택 가능 |

### 언어 감지 지원 언어 (12개)
한국어, 영어, 중국어, 일본어, 프랑스어, 스페인어, 베트남어, 태국어, 인도네시아어, 독일어, 러시아어, 이탈리아어

### API 호출 헤더 (시험 포인트!)

```http
X-NCP-APIGW-API-KEY-ID: {Client ID}      ← 앱 등록 시 발급받은 Client ID
X-NCP-APIGW-API-KEY: {Client Secret}     ← 앱 등록 시 발급받은 Client Secret
Content-Type: application/json
```

---

## 🔑 서비스 6: Search Trend — 네이버 검색 트렌드

### 특징

- 네이버 통합검색 데이터 기반 **검색어 통계 API**
- **RESTful API** 방식으로 제공
- 연령/성별/디바이스(PC/모바일)별 세분화 분석 가능
- 검색어 그룹 최대 **5개**, 그룹당 검색어 최대 **20개**

### 주요 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| `startDate` | string | Y | 시작일 (2016.01.01부터 제공) |
| `endDate` | string | Y | 종료일 |
| `timeUnit` | string | Y | `date` / `week` / `month` |
| `keywordGroups` | array | Y | 주제어 그룹 (최대 5개) |
| `device` | string | N | `pc` / `mo` |
| `gender` | string | N | `m` / `f` |
| `ages` | array | N | 나이 필터 (1~11 구간) |

### 나이 필터 구간

| 값 | 범위 |
|----|------|
| 1 | 0~12세 |
| 2 | 13~18세 |
| 3 | 19~24세 |
| 4 | 25~29세 |
| ... | ... |
| 11 | 60세 이상 |

---

## 🧠 AI 서비스 총정리 비교표

| 서비스 | 기능 | 방향 | 핵심 포인트 |
|--------|------|------|------------|
| **CLOVA Chatbot** | 대화형 AI | 텍스트↔텍스트 | 6개 언어, 앙상블 모델 |
| **CLOVA OCR** | 문서 인식 | 이미지→텍스트 | Basic/Premium 모델 구분 |
| **CLOVA Speech** | 음성 인식 | 음성→텍스트 (STT) | 부스팅 최대 500건 |
| **CLOVA Voice** | 음성 합성 | 텍스트→음성 (TTS) | 100가지 음성 옵션 |
| **Papago** | 번역 | 텍스트→텍스트 | 언어감지 12개 언어 |
| **Search Trend** | 검색 분석 | API→통계 | 그룹 5개, 그룹당 20개 |

---

## 시험 직전 체크리스트

- [ ] CLOVA Chatbot 지원 언어 6개 외우기
- [ ] 대화 우선순위: 객관식 버튼 > 컨텍스트 > 일반
- [ ] 네거티브 최대 20개 제한
- [ ] CLOVA OCR Basic vs Premium 차이 (멀티박스, 체크박스, 필드유형)
- [ ] TEXT OCR 경로 `/general`, Template OCR 경로 `/infer`
- [ ] Speech API 3가지 방식 (object-storage / url / upload)
- [ ] Speech 부스팅: 최대 500건, 한글·영어·숫자만, 1음절 불가
- [ ] CLOVA Voice 플랜: 실시간 API 호출 불가 (Standard/Premium 모두)
- [ ] Papago 언어감지: 12개 언어 지원
- [ ] Search Trend: 그룹 5개, 그룹당 키워드 20개

---

**참고 문서**:
- [CLOVA Chatbot 개요](https://guide.ncloud-docs.com/docs/chatbot-chatbot-overview)
- [CLOVA OCR 개요](https://guide.ncloud-docs.com/docs/clovaocr-overview)
- [CLOVA Voice 개요](https://guide.ncloud-docs.com/docs/clovavoice-overview)
- [Search Trend 개요](https://guide.ncloud-docs.com/docs/searchtrend-overview)
