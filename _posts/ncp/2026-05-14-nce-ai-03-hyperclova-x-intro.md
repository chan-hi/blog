---
title: "[NCE-AI] 03. HyperCLOVA X 소개"
date: 2026-05-14 09:30:00 +0900
categories: [NCP, NCE-AI]
tags: [HyperCLOVA X, NCP, LLM, Vision, CLOVA Studio, NCE-AI]
description: "HyperCLOVA X 한국어 특장점, 서비스 모델(Basic/Exclusive/Neurocloud), Vision 기능, HCX-DASH Binary 정리"
---

## 핵심 요약

- HyperCLOVA X는 한국어와 한국 문화 맥락에 강점을 가진 초거대 언어 모델입니다.
- 한국어 벤치마크, 토큰 효율성, 한국어 문장 구조 이해, 다국어 능력, Vision 기능이 강조됩니다.
- CLOVA Studio는 Basic, Exclusive, Neurocloud 형태로 제공되며, 고객의 보안/성능/비용 요구에 따라 선택합니다.
- HCX-DASH 계열은 경량 모델이며, HCX-DASH Binary는 고객 인프라에 경량 모델 파일을 제공하는 방식입니다.

## HyperCLOVA X의 한국어 특장점

### KorNAT와 한국어 평가

HyperCLOVA X는 한국어 문장의 맥락과 한국 고유 지식을 이해하는 데 강점이 있습니다. 원문에서는 영어권 문화에 적절한 답변이 한국 문화에서는 부적절할 수 있으므로, 한국어를 평가할 별도 지표가 필요하다고 설명합니다.

핵심 내용은 다음과 같습니다.

- 한국어 적합도 측정을 위해 `KorNAT` 지표를 만들었습니다.
- 한국정보통신기술협회, TTA의 정성적/정량적 표준 평가를 통과해 정부 승인을 받았습니다.
- HyperCLOVA X는 특히 국어와 한국사에서 높은 점수를 보였고, 수학과 과학을 제외한 대부분의 과목에서 다른 모델보다 우수한 성능을 보였습니다.

### KMMLU

`KMMLU`는 한국어 언어 모델 벤치마크입니다. 정식 의미는 `Measuring Massive Multitask Language Understanding in Korean`입니다.

- 45개 주제
- 35,030개의 전문 객관식 질의응답
- 수학, 물리, 역사, 법률, 의학, 윤리 등 다양한 분야의 전문 지식 평가
- STEM은 Science, Technology, Engineering, Math의 약자

시험에서는 KMMLU가 한국어 기반 전문지식 평가 벤치마크라는 점을 기억하면 됩니다.

### 다국어 능력과 토큰 효율성

HyperCLOVA X는 번역 성능과 한국어 특화 토크나이저를 장점으로 제시합니다.

| 번역 방향 | HyperCLOVA X | 비교 모델/번역기 |
|---|---:|---:|
| 영어 -> 한국어 | 93.8 | 89.0, 89.1 |
| 일본어 -> 한국어 | 90.3 | 90.2, 83.3 |
| 중국어 -> 한국어 | 88.8 | 88.8, 86.2 |

한국어 특화 토크나이저를 활용하면 해외 LLM 대비 최고 2배까지 빠르고 저렴할 수 있다고 설명합니다.

### 한국 문화와 맥락 이해

원문에서는 지오디의 `거짓말` 가사처럼 겉말과 속마음이 다른 한국어 표현을 예시로 듭니다. HyperCLOVA X는 문장 속 괄호 표현과 반어적 맥락을 이해하여, 단순히 `가라`가 아니라 `가지 말라고 말리는 내용`으로 해석할 수 있음을 보여줍니다.

### 한국어 문장 구조 이해

한국어는 조사, 명사, 어미, 어간이 결합하면서 다양한 의미를 만듭니다.

| 요소 | 예 | 의미 |
|---|---|---|
| 조사 | 은/는, 이/가, 을/를, 보다, 마저 | 단어와 문장 성분의 관계 표시 |
| 명사 | 사람, 사물, 동물 이름 | 대상 이름 |
| 어미 | 로서/로써, 러/려, 으므로/음으로, 대요/데요 | 문장 끝이나 활용 의미 변화 |
| 어간 | `산뜻하-` | 동사/형용사에서 변하지 않는 부분 |

HyperCLOVA X는 한국어 문장 토큰화와 구조 이해를 통해 자연스러운 문장을 생성하는 모델로 소개됩니다.

## HyperCLOVA X 라인업과 활용

원문은 사용자, 사업자, 기업/공공기관을 대상으로 다양한 서비스 라인업을 제시합니다.

| 대상 | 예시 |
|---|---|
| 사용자 | CLOVA X, cue:, 캐릭터챗 |
| 사업자 | CLOVA for Writing, CLOVA for AD, CLOVA for 스마트스토어센터, CLOVA for 스마트플레이스 |
| 기업/공공기관 | CLOVA Studio, CLOVA Studio CSAP, HyperCLOVA X 구축형 모델, Neurocloud for HyperCLOVA X |

예시 화면으로는 생성형 AI 검색 `cue:`, 대화형 AI 서비스 `CLOVA X`, AI 글쓰기 도구 `CLOVA for Writing`, 스마트 광고 `CLOVA for AD`, 스마트스토어 상품 소개 초안 생성, CLOVA Studio 플레이그라운드, CONNECT X 업무 지원 화면이 제시됩니다.

## CLOVA Studio 서비스 모델

### 전체 비교

| 구분 | 특징 | 적합한 상황 |
|---|---|---|
| Basic | 저렴한 종량제, 별도 계약 없이 사용 신청으로 빠르게 이용 | 테스트, 빠른 개발, 사용량 기반 과금 |
| Exclusive | 고객별 전용 인프라와 GPU 할당, TPM 보장, 월 정액 | 안정적인 상용 서비스, 전용 모델 필요 |
| Neurocloud | 고객사 내부에 설치된 하이브리드 클라우드, 전용 HW/SW | 강한 보안 규제, 고객 IDC 내 설치 필요 |
| HCX-DASH Binary | HCX-DASH 경량 모델 Binary 제공 | 고객 보유 GPU 인프라에서 자체 운영 |

### Basic

Basic은 CLOVA Studio 신청만으로 이용할 수 있습니다. 신규 모델, 기능, 도구가 빠르게 제공되며, 토큰 사용량에 따른 종량제 과금입니다.

주요 특징은 다음과 같습니다.

- 빠른 scale-in/out
- 낮은 진입장벽
- 저렴한 사용 비용
- 신속한 신규 모델 사용
- Playground, Tuning, Explorer, Skill Trainer, API 사용 가능

### Exclusive

Exclusive는 고객별 모델과 GPU를 할당하여 서비스 안정성을 확보합니다.

- HCX-005, HCX-DASH-002 모델을 고객별로 별도 제공
- Inference TPM 보장
- Basic과 동일한 기본 기능 제공
- 월 정액/구독형 과금
- 기본 튜닝 학습용 무료 크레딧은 별도 협의
- SFT, FP, RLHF 기반 Advanced Tuning 제공 가능
- 인퍼런스 분리를 통한 보안 강화

### Neurocloud for HyperCLOVA X

Neurocloud는 고객사 IDC 내부에 설치하여 보안 요구를 만족시키는 형태입니다.

- 고객사 전용 HW/SW 제공
- 완전관리형 서비스
- 모든 데이터가 사내망 안에서 처리
- 네이버클라우드와 전용선으로 연결된 영역과 사내망을 분리
- 하드웨어 장애관리, SW/모델 업데이트 지원
- SFT/FP/RLHF 가능 여부에 따라 Type A/B 제안

## HyperCLOVA X Vision

HyperCLOVA X Vision은 기존 언어 모델에 이미지 이해 능력을 더한 거대 시각 언어 모델, Large Vision Language Model입니다.

### 주요 작업

| 작업 | 설명 |
|---|---|
| Detailed Image Captioning | 이미지의 세부 내용을 비교적 정확히 인식하고 묘사 |
| Entity Recognition | 인명, 장소, 제품 등 의미 있는 단위를 이미지에서 인식 |
| Reasoning | 이미지 이해를 바탕으로 상황을 추론하고 다음 단계를 예측 |
| Creative Writing with Image Grounding | 이미지 요소를 기반으로 창의적인 글쓰기 |
| Chart, Table, Document Understanding | 이미지 속 텍스트와 위치 관계를 이해 |
| Culture and Humor, Meme Understanding | 이미지와 텍스트 쌍 학습을 바탕으로 문화/유머/밈 이해 |

### VLM 활용 예시

VLM, Vision-Language Model은 비전과 언어 양식을 통합하는 모델입니다. 원문에서 제시한 활용 예시는 다음과 같습니다.

- 회의록 스케치, 도표, 구조도 인식
- 다이어그램의 데이터 처리 흐름 설명
- 상품 상세 페이지에서 설명 추출 및 요약
- 상품 관련 질의에 RAG 방식으로 답변
- 테이블/문서 이미지 내용을 이해하고 요약

## HCX-DASH Binary

HCX-DASH Binary는 HCX-DASH 경량 모델을 Binary 파일 형태로 제공하는 방식입니다.

| 구분 | Basic | Exclusive | Binary |
|---|---|---|---|
| 운영 위치 | NAVER Cloud Infra | NAVER Cloud Infra, 전용 GPU | 고객 보유 GPU 인프라 |
| 과금 | 토큰 사용량 종량제 | 월 구독형 정액제 | 사용 계약 기반 |
| Rate Limit | Shared GPU Pool 기반이라 보장 없음 | Dedicated GPU 기반으로 보장 | 고객 구현 영역 |
| 제공 모델 | HCX 계열 | HCX 계열 | HCX-DASH 경량 모델 only |

## 시험 암기 포인트

- HyperCLOVA X의 차별점은 한국어, 한국 문화, 한국 지식, 한국어 토크나이저입니다.
- `KMMLU`는 한국어 전문지식 벤치마크입니다.
- `Basic`은 종량제와 빠른 사용, `Exclusive`는 전용 GPU와 TPM 보장, `Neurocloud`는 고객 IDC 내부 설치가 핵심입니다.
- Vision은 이미지 설명, 객체/장소/제품 인식, 추론, 문서/테이블 이해, 밈 이해까지 포함합니다.
- `HCX-DASH Binary`는 고객 인프라에서 경량 모델을 운영하는 계약 기반 제공 방식입니다.

## 원문 반영 체크

- 반영 페이지: 31~54p
- 포함 내용: KorNAT, KMMLU, 번역 성능, 한국어 맥락/문장 구조, 라인업, CLOVA X 및 관련 서비스 예시, CLOVA Studio Basic/Exclusive/Neurocloud, Vision 기능, VLM 활용, HCX-DASH Binary
