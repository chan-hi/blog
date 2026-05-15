---
title: "[NCE] 09. AI & Application 서비스"
date: 2026-05-14 10:20:00 +0900
categories: [NCP, NCE]
tags: [NCP, NCE, CLOVA, OCR, Chatbot, Papago, AiTEMS, Geolocation, API Gateway, 인공지능]
description: "CLOVA Speech/Voice/OCR/Chatbot/Dubbing, AiTEMS 추천, Papago, API 인증, Outbound Mailer, Geolocation, Search Trend API 정리"
---

## 핵심 요약

- CLOVA Speech: REST 동기 최대 2h, 배치 비동기 최대 6h, gRPC 최대 100h
- CLOVA Voice: 100가지 목소리, 최대 2,000자
- CLOVA OCR: Basic(활자체) vs Premium(활자체+필기체, 멀티박스, 체크박스)
- CLOVA Chatbot: 라인/톡톡/페이스북 연동, 네거티브 최대 20개
- CLOVA Dubbing: 동영상 최대 20분/500MB
- AiTEMS: 인기/개인화/연관 3가지 추천 모델
- Papago: 12개 언어 지원
- Outbound Mailer: 콘솔 최대 1,000명 / 파일 100,000명
- Geolocation: 국내 동(洞)까지, 해외 주(州)까지

## API Gateway 인증

API 호출 시 필수 헤더:

| 헤더 | 설명 |
|------|------|
| `x-ncp-apigw-timestamp` | 요청 타임스탬프 |
| `x-ncp-iam-access-key` | IAM Access Key |
| `x-ncp-apigw-signature-v2` | HmacSHA256 서명 |

## CLOVA Speech (STT)

음성 → 텍스트 변환 서비스.

### 인식 방식 비교

| 방식 | 최대 시간 | 특징 |
|------|----------|------|
| **REST (동기)** | **2시간** | 짧은 음성, 즉시 결과 반환 |
| **REST (배치 비동기)** | **6시간** | 긴 파일, 비동기 처리 |
| **gRPC (스트리밍)** | **100시간** | 실시간 스트리밍 인식 |

### 주요 기능

- 도메인 특화 언어 모델 지원
- **키워드 부스팅**: 특정 단어 인식률 향상, 최대 **500개** 등록
- 발화자 분리(Diarization): 여러 화자 구분

## CLOVA Voice (TTS)

텍스트 → 음성 합성 서비스.

- **100가지** 목소리 지원
- **최대 2,000자** 변환
- 음성 파라미터: 말하기 속도, 음높이, 볼륨 조절

## CLOVA OCR

이미지/문서에서 텍스트 추출 서비스.

### OCR 종류 비교

| 항목 | Basic | Premium |
|------|-------|---------|
| **활자체** 인식 | 지원 | 지원 |
| **필기체** 인식 | **미지원** | **지원** |
| **멀티박스** | **미지원** | **지원** |
| **체크박스** | **미지원** | **지원** |

### 특수 OCR

| 서비스 | 설명 |
|--------|------|
| **Document OCR** | 신분증(여권/주민증), 사업자등록증 구조화 추출 |
| **Table OCR** | 표 구조 인식 및 데이터 추출 |
| **Medical OCR** | 진단서, 처방전 등 의료 문서 특화 |
| **Template OCR** | 사용자 정의 템플릿 기반 추출 |
| **PDF OCR** | PDF 문서 텍스트 추출 (Layer 없는 이미지형 PDF 포함) |

## CLOVA Chatbot

AI 기반 챗봇 구축/운영 서비스.

### 연동 채널

- **LINE, 네이버 톡톡, Facebook Messenger**

### 주요 개념

| 개념 | 설명 |
|------|------|
| **Intent** | 사용자 발화 의도 분류 단위 |
| **Entity** | 발화 내 핵심 정보 추출 단위 |
| **네거티브** | 무응답 처리할 학습 문장, 최대 **20개** |
| **폴백(Fallback)** | 매칭 Intent 없을 때 기본 응답 |

## CLOVA Dubbing

동영상 자동 더빙 서비스.

- **동영상 최대 20분** / **500MB**
- 텍스트 스크립트 → 음성 합성 → 더빙 영상 생성
- 다국어 더빙 지원

## CLOVA Studio

초거대 AI(HyperCLOVA X) 기반 텍스트 생성 서비스.

- 문서 요약, 감성 분석, 텍스트 분류, 문장 생성
- Task Tuning으로 도메인 특화 모델 생성 가능

## AiTEMS (추천 AI)

AI 기반 개인화 추천 서비스.

### 3가지 추천 모델

| 모델 | 설명 |
|------|------|
| **인기 추천** | 인기 아이템 기반 추천 |
| **개인화 추천** | 사용자 행동 데이터 기반 맞춤 추천 |
| **연관 추천** | 조회/구매 아이템과 연관된 상품 추천 |

- 클릭/구매/장바구니 데이터 학습
- A/B 테스트 기능 제공

## Papago (번역)

AI 기반 자동 번역 서비스.

- **12개 언어** 지원: 한국어, 영어, 일본어, 중국어(간/번체), 프랑스어, 스페인어, 베트남어, 태국어, 인도네시아어, 독일어, 러시아어
- 텍스트/웹사이트/파일 번역
- 전문 용어 커스텀 사전 등록 가능

## Outbound Mailer

대규모 이메일 발송 서비스.

| 항목 | 값 |
|------|-----|
| **콘솔 직접 등록** | 최대 **1,000명** |
| **파일 업로드** | 최대 **100,000명** |
| **발송 방식** | **30명씩** 순차 발송 |

- 발송 결과 통계 (수신/열람/클릭율)
- 수신거부 관리

## Geolocation

IP 주소 기반 위치 정보 조회 서비스.

| 지역 | 정밀도 |
|------|--------|
| **국내** | **동(洞)** 단위까지 |
| **해외** | **주(州)** 단위까지 |

- IP 입력 → 위도/경도, 국가/도시 반환
- 접근 제한, 맞춤 서비스에 활용

## Search Trend API (데이터랩)

검색 트렌드 데이터 제공 API.

| 항목 | 값 |
|------|-----|
| **일일 최대 호출** | **1,000건** |
| **최대 그룹 수** | **5개** |
| **그룹당 검색어** | 최대 **20개** |
| **연령대 코드** | `ages` 1~11 |

## 시험 암기 포인트

- **CLOVA Speech**: REST 동기 2h / 배치 비동기 6h / gRPC 100h / 키워드 부스팅 500개
- **CLOVA Voice**: 100가지 목소리, 최대 2,000자
- **OCR Basic vs Premium**: Premium = 필기체 + 멀티박스 + 체크박스
- **CLOVA Chatbot**: LINE/톡톡/Facebook 연동, 네거티브 최대 20개
- **CLOVA Dubbing**: 최대 20분/500MB
- **AiTEMS**: 인기/개인화/연관 3가지 추천 모델
- **Papago**: 12개 언어
- **Outbound Mailer**: 콘솔 1,000명 / 파일 100,000명 / 30명씩 발송
- **Geolocation**: 국내 동(洞)까지 / 해외 주(州)까지
- **Search Trend API**: 일 1,000건, 최대 5그룹, 그룹당 20개, ages 1~11
- **API 인증**: x-ncp-apigw-timestamp + x-ncp-iam-access-key + x-ncp-apigw-signature-v2 (HmacSHA256)

