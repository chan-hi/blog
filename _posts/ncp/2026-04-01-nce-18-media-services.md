---
title: "Chapter 18: 빵집 생방송 — 미디어 서비스"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, media, live-station, vod-station, image-optimizer]
description: "NCP NCE 자격증 대비 학습 자료 - Live Station, VOD Station, Image Optimizer 핵심 정리"
---
> **학습 목표**: Live Station 입출력 규격/제한사항, VOD Station, Image Optimizer, Video Player 특징을 정확히 이해한다.

---

## 이야기: 클라우드빵집 유튜브 채널 개설

클라우드빵집이 빵 만드는 과정을 생방송으로 보여주기로 했다.
또한 수십만 장의 상품 이미지를 모바일에서 빠르게 로드해야 했다.

> "라이브 방송 서버를 직접 구축해야 해?"

재현이 답했다.

> "NCP에 Live Station이랑 VOD Station이 있어.
> 그리고 이미지 최적화는 Image Optimizer 쓰면 돼."

---

## 🔑 핵심 개념 1: Live Station (라이브 스테이션)

단일 RTMP 스트림 → 여러 해상도 Multi-bitrate → HLS로 패킷화 전송.

### 입출력 규격

| 항목 | 규격 |
|------|------|
| **Input 프로토콜** | **RTMP** |
| **Input Video 코덱** | H.264, H.265 |
| **Input Audio 코덱** | AAC, AAC-LC |
| **Output 프로토콜** | **HLS, LL-HLS, MPEG-DASH** |
| **동시 송출 추가** | RTMP (H.264, AAC 지원) |

### 제한 사항 (⭐ 필수 암기!)

| 항목 | 제한 |
|------|------|
| **최대 해상도** | **1920×1080@60fps** |
| **채널당 동시 변환** | 최대 **5개** |
| **최대 비트레이트** | **20Mbps** |
| **CDN 연동 동시 접속자** | **1만 명** 지원 (초과 시 사전 협의) |

### 부가 기능

- **Time Shift**: 라이브 방송을 되감아 볼 수 있는 기능
- **실시간 DVR**: MP4, HLS 형태로 실시간 녹화
- **Live Thumbnail**: 썸네일 자동 생성
- **Re-Stream**: 타 플랫폼으로 동시 송출
- **VOD2LIVE**: 녹화된 VOD를 라이브처럼 재생

---

## 🔑 핵심 개념 2: VOD Station (VOD 스테이션)

VOD 영상을 다양한 환경(모바일, PC)에 맞게 변환하여 제공.

```
원본 영상 (Object Storage)
      ↓
VOD Station (nginx-vod-module 사용)
      ↓ 패킷타이징 (일정 간격으로 파일 분할)
HLS / DASH 스트리밍
      ↓
CDN → 사용자
```

### 주요 기능

| 기능 | 설명 |
|------|------|
| **인코딩 템플릿** | 자주 쓰는 인코딩 옵션을 템플릿으로 저장 후 재사용 |
| **One Click Multi DRM** | Widevine, FairPlay, PlayReady 원클릭 구축 |
| **썸네일 추출** | 인코딩 중 고품질 썸네일 자동 추출 → CDN 연동 |

### 스트리밍 URL 포맷

```
https://[cdnDomain]/[protocol]/[path]/[video filename]/[manifest]
```

---

## 🔑 핵심 개념 3: Image Optimizer (이미지 옵티마이저)

**Object Storage 원본 이미지**를 쿼리스트링 변환 룰로 **실시간 변환 후 CDN 제공**.

```
원본: https://storage.../cake.jpg (5MB, 4000×3000)
요청: https://cdn.../cake.jpg?type=m&width=400&height=300&quality=80
결과: 실시간으로 400×300, 80% 품질로 변환 후 응답

→ 모바일에 최적화된 이미지 즉시 제공!
```

### 주요 전역 옵션 파라미터

| 파라미터 | 역할 |
|---------|------|
| `quality` | JPEG 품질 설정 (1~100) |
| `autorotate` | EXIF 정보 기준 자동 회전 (true/false) |
| `anilimit` | GIF 노출 프레임 개수 제한 |
| `ttype` | 출력 포맷 지정 (jpg, gif, png) |

---

## 🔑 핵심 개념 4: Video Player Enhancement

웹/앱용 **커스터마이징 플레이어 서비스 (SaaS 형태)**.

| 특징 | 내용 |
|------|------|
| **설치 방식** | 별도 앱 설치 불필요 (**HTML5 표준**) |
| **관리** | 콘솔 기반 UI/UX 패널, 스크립트 코드 자동 생성 |
| **보안** | Multi DRM, 워터마크 지원 |
| **모바일** | Native SDK (Android, iOS) 제공 |
| **저지연** | **CMAF LL-HLS** (Low Latency) 지원 |
| **대역폭** | 기본 재생 대역폭: **1, 2, 5Mbps** 설정 가능 |

---

## 🔑 핵심 개념 5: 통합 관리 서비스

| 서비스 | 역할 |
|--------|------|
| **Media Connect Center** | Object Storage, VOD, Live Station 등 워크플로우를 **한 화면에서 통합 관리** (검색, 분류, 권한 보안) |
| **Media AI Understanding** | 비전 AI + 음성 AI 결합. 영상 내 인물/객체/행동 분석, **자동 자막** 추출 |

---

## 📝 시험 대비 체크리스트

- [ ] Live Station Input: **RTMP** / Output: **HLS, LL-HLS, MPEG-DASH**
- [ ] Live Station 최대 해상도: **1920×1080@60fps**
- [ ] Live Station 채널당 동시 변환: **최대 5개**
- [ ] Live Station 최대 비트레이트: **20Mbps**
- [ ] Live Station CDN 동시 접속자: **1만 명**
- [ ] VOD Station: `nginx-vod-module`, Multi DRM, 썸네일 추출
- [ ] Image Optimizer: Object Storage 원본 → 실시간 변환 → CDN
- [ ] Image Optimizer `quality`: JPEG 품질 / `ttype`: 출력 포맷
- [ ] Video Player: HTML5 표준, SaaS, CMAF LL-HLS
- [ ] Media Connect Center: 워크플로우 통합 관리

---

## 🧠 암기 핵심 문장

> "**Live Station: RTMP 입력, HLS 출력, 1080p@60fps, 5채널, 20Mbps**"  
> "**Image Optimizer = Object Storage 원본 → 쿼리스트링으로 실시간 변환**"

---

*다음 챕터: `19_데이터베이스_MySQL_MSSQL.md` → 클라우드 데이터베이스 서비스*
