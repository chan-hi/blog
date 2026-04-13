---
title: "Chapter 18: 빵집 생방송 — 미디어 서비스"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, media, live-station, vod-station, image-optimizer]
description: "NCP NCE 자격증 대비 학습 자료 - Live Station, VOD Station, Image Optimizer 핵심 정리"
---
> **학습 목표**: Live Station 입출력 규격/제한사항, VOD Station, Image Optimizer, Video Player Enhancement, Media Connect Center, Media AI Understanding의 특징을 정확히 이해한다.

---

## 이야기: 클라우드빵집 유튜브 채널 개설

클라우드빵집이 빵 만드는 과정을 생방송으로 보여주기로 했다.
또한 수십만 장의 상품 이미지를 모바일에서 빠르게 로드해야 했다.

> "라이브 방송 서버를 직접 구축해야 해?"

재현이 답했다.

> "NCP에 Live Station이랑 VOD Station이 있어.
> 이미지 최적화는 Image Optimizer 쓰면 되고,
> 전체 미디어 워크플로우는 Media Connect Center로 한 화면에서 다 관리할 수 있어."

---

## 핵심 개념 1: Live Station (라이브 스테이션)

실시간 방송을 위한 플랫폼. 단일 RTMP 스트림을 여러 개의 Multi-bitrate 출력 스트림으로 변환하고, 영상을 HLS로 패킷화하여 전송한다. CDN(Global Edge) 연동을 통해 안정적인 송출이 가능하다.

```
단일 RTMP 원본 스트림
      ↓ 트랜스코딩 (멀티 화질 변환)
Multi-bitrate 출력 스트림
      ↓ HLS 패킷화
HLS / LL-HLS / MPEG-DASH
      ↓ CDN(Global Edge)
시청자 (동시 1만 명 지원)
```

### 입출력 규격

| 항목 | 입력(Input) | 출력(Output) |
|------|------------|-------------|
| **프로토콜** | RTMP | HLS, LL-HLS, MPEG-DASH, RTMP(동시 송출 전용) |
| **비디오 코덱** | H.264, H.265 | H.264 |
| **오디오 코덱** | AAC, AAC-LC | AAC |
| **비트레이트** | 최대 20Mbps | HLS Adaptive Bitrate 지원 |

### 제한 사항 (⭐ 필수 암기!)

| 항목 | 제한 |
|------|------|
| **최대 해상도** | **1920×1080@60fps** |
| **채널당 동시 변환** | 최대 **5개** |
| **최대 비트레이트** | **20Mbps** |
| **CDN 연동 동시 접속자** | **1만 명** 지원 (초과 시 사전 협의) |

### 채널 상세 정보

채널을 생성하면 아래 항목을 콘솔에서 확인할 수 있다.

| 항목 | 설명 |
|------|------|
| **CDN(Global Edge) 적용** | 선택한 채널의 CDN 적용 여부 |
| **채널 삭제 예정일** | 최근 30일간 입력이 없었던 미사용 채널은 자동 반납 예정일 표시 |
| **트랜스코딩 설정** | 선택한 채널에 적용된 트랜스코딩 상세 설정 확인 |
| **Push URL** | 방송 진행을 위해 인코더에 입력·송출해야 할 RTMP URL |
| **Play URL** | ABR(Adaptive Bit Rate)로 트랜스코딩 설정이 반영된 재생 URL. 사용자 네트워크 대역폭이 높으면 HD, 1Mbps 초과 시 360p 화질로 재생 |
| **Thumbnail URL** | 송출 중인 라이브 영상에서 추출한 캡처 이미지 다운로드 URL |
| **Recording URL** | Live Transcoder 원본 영상 보관·확인 URL (NCP Storage 보유 사용자 대상 부가 기능) |

### 부가 기능

| 기능 | 설명 |
|------|------|
| **Time Shift** | 라이브 방송을 되감아 볼 수 있는 타임머신 기능 |
| **실시간 DVR** | MP4, HLS 형태로 실시간 녹화(Digital Video Recorder) |
| **Live Thumbnail** | 썸네일 이미지 자동 추출 |
| **Re-Stream** | 타 플랫폼으로 동시 송출. YouTube, Twitch, Mixer, Periscope, Steam, Ustream, Younow, Afreeca, NAVER, VLIVE, LINE TV, Prism, Custom 등 지원 |
| **VOD2LIVE** | 녹화된 VOD를 라이브처럼 재생 |
| **스트림 모니터링** | 스트림 상태를 실시간으로 확인하는 모니터링 기능 제공 |

---

## 핵심 개념 2: VOD Station (VOD 스테이션)

VOD 동영상을 다양한 시청 환경(모바일, PC)에 맞게 변환하여 송출하는 서비스. 패킷타이징을 위해 **nginx-vod-module**을 사용한다.

> 패킷타이징이란 스트림으로 출력 시 파일을 일정 간격으로 분할하여 송출하는 것. 분할 간격이 너무 크면 실시간성은 떨어지지만, 버퍼링을 통해 안정적인 스트리밍이 가능하다.

```
원본 영상 (Object Storage)
      ↓
VOD Station (nginx-vod-module)
      ↓ 패킷타이징 (일정 간격으로 파일 분할)
HLS / DASH 스트리밍
      ↓
CDN → 사용자
```

### 주요 기능

| 기능 | 설명 |
|------|------|
| **인코딩 템플릿** | 비즈니스·콘텐츠 분야별 기본 인코딩 설정 목록을 템플릿으로 제공. 복잡한 옵션을 따로 설정하지 않고 목록에서 선택하여 손쉽게 사용 |
| **One Click Multi DRM** | Widevine, FairPlay, PlayReady 원클릭 구축 (Pallycon DRM CPIX v1 API) |
| **썸네일 추출** | 비디오 인코딩 작업 중 원본 영상에서 고품질 썸네일 이미지 자동 추출. 인코딩 파일과 다른 스토리지 경로에 저장 가능하며 CDN과 연동하여 서비스에 활용 |

### 지원 사양

**Input 규격**

| 항목 | 지원 |
|------|------|
| **컨테이너 형식** | AVI, MOV, MP4, MP3, 3GP, MPG, MPEG, M4V, VOB, WMV, ASF, MKV, FLV, WEBM, Animated GIF, AV1, MXF |
| **비디오 코덱** | H.264, VP9, VP8, MPEG-2 |
| **오디오 코덱** | AAC, MP3, MP2, PCM, FLAC, Vorbis |
| **파일 크기 제한** | 없음 |
| **자막 입력 형식** | vtt, srt, dfxp, ttml, cap |

**Output 규격**

| 항목 | 지원 |
|------|------|
| **스트리밍 프로토콜** | HLS, DASH (Adaptive Bitrate 지원) |
| **컨테이너 형식** | MP4, M4A |
| **비디오 코덱** | H.264 |
| **오디오 코덱** | AAC |
| **세그먼트 길이(초)** | 5, 10, 15, 20 |
| **자막 출력 형식** | vtt |
| **해상도** | 2160p(UHD), 1080p(FHD), 720p(HD), 480p(SD) |
| **썸네일 이미지** | 최대 10개 |
| **다중 출력** | 최대 5개 |
| **추가 옵션** | 비디오 전용, 오디오 전용, 자르기(Cropping), Channel DRM Packaging |

### 스트리밍 URL 포맷

```
https://[cdnDomain]/[protocol]/[path]/[video filename]/[manifest]
```

예시:
- HLS: `https://example.cdn.ntruss.com/hls/vod-5100k.mp4/index.m3u8`
- DASH: `https://example.cdn.ntruss.com/dash/vod-5100k.mp4/manifest.mpd`

---

## 핵심 개념 3: Image Optimizer (이미지 옵티마이저)

Object Storage에 업로드된 원본 이미지를 미리 설정된 룰(쿼리스트링)에 따라 실시간으로 변환하고 CDN(Global Edge)을 통해 제공하는 서비스. 리사이즈, 크롭 시 자동 회전, 얼굴 인식 기능을 제공하며, 변환 룰은 쿼리스트링으로 작성되어 개발 코드에 그대로 적용함으로써 개발 시간을 단축한다.

```
원본: https://storage.../cake.jpg (5MB, 4000×3000)
요청: https://cdn.../cake.jpg?type=m&width=400&height=300&quality=80
결과: 실시간으로 400×300, 80% 품질로 변환 후 응답

→ 모바일에 최적화된 이미지 즉시 제공!
```

### 설정 절차

1. Object Storage 버킷에 원본 이미지 저장
2. Image Optimizer에서 버킷 선택 및 Project 생성
3. Project에 적용될 변환 허용 타입 설정
4. CDN 설정 및 Cloud Log Analytics 연동 신청

### 전역 옵션 파라미터

| 파라미터 | 역할 |
|---------|------|
| `quality` | 원본이 JPEG인 경우에만 적용. 1~100 범위의 정수 (기본값: 90) |
| `autorotate` | EXIF 회전 정보를 토대로 결과 이미지를 자동 회전 (기본값: false) |
| `anilimit` | GIF에 여러 프레임이 존재할 경우 앞에서부터 노출할 프레임 개수 1~100 (기본값: 1) |
| `ttype` | 출력 포맷 지정 (jpg, gif, png). 설정하지 않으면 원본 포맷과 동일 |

### 변환 옵션

| 옵션 | 설명 |
|------|------|
| **리사이즈 및 크롭** (`type=f`) | 원본 비율을 유지하면서 이미지 크기를 축소·확대한 후 남는 영역 크롭 |
| **리사이즈** (`type=m`) | 설정한 가로·세로 크기 안에 들어오도록 이미지 크기 변경 |
| **워터마크** (`type=wm`) | 이미지에 워터마크 적용 |
| **샤픈(Sharpen)** | 이미지 선명도 향상 |
| **블러(Blur)** | 이미지 흐림 효과 적용 |

### 연동 구조

변환 이력은 Cloud Log Analytics에 저장되며, Cloud Log Analytics를 통해 변환 이력을 조회할 수 있다.

---

## 핵심 개념 4: Video Player Enhancement

웹/앱에서 비디오 또는 오디오와 같은 미디어를 재생할 수 있는 NCP의 **SaaS 형태** 플레이어 서비스.

| 특징 | 내용 |
|------|------|
| **설치 방식** | 별도 앱 설치 불필요 (**HTML5 표준**). 모든 디바이스 및 OS 브라우저에서 재생 가능 |
| **플레이어 커스터마이징** | 기본 제공 플레이어에 원하는 옵션을 설정하여 커스터마이징 가능 |
| **관리** | 콘솔 기반 UI/UX 패널로 직접 플레이어 코드를 수정하지 않고 손쉽게 옵션 설정. 스크립트 코드 자동 생성 |
| **보안** | Multi DRM (FairPlay, Widevine, PlayReady), Visible 워터마크 지원 |
| **모바일** | Native SDK (AOS-Kotlin, iOS-Swift) 제공 |
| **저지연** | **CMAF LL-HLS** (Low Latency HLS) 지원 |
| **대역폭** | 기본 재생 대역폭: **1Mbps, 2Mbps, 5Mbps** 설정 가능. Adaptive Streaming 서비스 구현 시 유용 |

---

## 핵심 개념 5: Media Connect Center (미디어 커넥트 센터)

Object Storage, VOD Station, Live Station과 결합하여 **미디어 파일 관리를 위한 그룹화, 인코딩, CDN 연동, 채널, 배포까지 일원화된 사용성**을 통해 미디어 서비스 전체를 **한 화면에서 통합 관리·운영**할 수 있는 서비스.

| 기능 | 설명 |
|------|------|
| **상세 검색 및 분류** | 콘텐츠 메타정보를 입력하는 상세 검색으로 원하는 내용을 빠르게 검색. 미디어 서비스별 콘텐츠를 분류하여 체계적 관리 |
| **미디어 상품 간편 연동** | 미디어 상품별로 이원화된 워크플로우를 통합. Object Storage 업로드 → CDN 연동 → 인코딩 → 채널 생성 → 배포까지 일원화하여 최상의 미디어 운영 환경 제공 |
| **강력한 보안** | 콘텐츠에 접근하는 구성원의 권한 구분 가능. Object Storage의 보안 설정과 연동하여 외부 불법 콘텐츠 유출에 견고하게 대처 |

---

## 핵심 개념 6: Media AI Understanding

비전 분석 AI와 음성 분석 AI를 통합한 **멀티모달 모델 엔진**을 활용하여 영상 및 음성 분석 결과를 종합적으로 이해하는 서비스.

- **인물 분석**: 영상 내 인물 자동 인식 및 분류
- **행동/객체/시공간 분석**: 영상 속 객체·행동·장면 자동 분석
- **자동 자막**: 음성 분석 AI를 통한 자막 자동 추출

---

## 실습 항목 (Lab 10. Media 상품 실습)

교재 실습에서는 다음 항목을 직접 수행한다:

1. CLI로 로드 밸런서 생성
2. 라이브 스테이션 영상 저장을 위한 Object Storage 버킷 생성
3. 라이브 스테이션 구성
4. VOD 스테이션을 이용하여 라이브 스테이션으로 녹화된 영상 VOD 서비스
5. Media Connect Center를 이용하여 영상 관리
6. Image Optimizer 이미지 업로드용 Object Storage 버킷 생성
7. Image Optimizer로 변환 룰 생성 후 이미지 차이 확인

---

## 시험 대비 체크리스트

- [ ] Live Station Input: **RTMP** / Output: **HLS, LL-HLS, MPEG-DASH, RTMP(동시 송출)**
- [ ] Live Station 최대 해상도: **1920×1080@60fps**
- [ ] Live Station 채널당 동시 변환: **최대 5개**
- [ ] Live Station 최대 비트레이트: **20Mbps**
- [ ] Live Station CDN 동시 접속자: **1만 명** (초과 시 협의)
- [ ] Live Station Play URL: ABR 지원, 1Mbps 이상 360p 재생
- [ ] Live Station 미사용 채널: **30일** 무입력 시 자동 반납
- [ ] VOD Station: `nginx-vod-module`, Multi DRM (Pallycon), 썸네일 추출
- [ ] VOD Station 세그먼트 길이: **5, 10, 15, 20초**
- [ ] VOD Station 다중 출력: 최대 **5개**, 썸네일 최대 **10개**
- [ ] VOD Station 해상도: UHD(2160p), FHD(1080p), HD(720p), SD(480p)
- [ ] Image Optimizer: Object Storage 원본 → 쿼리스트링 룰 → 실시간 변환 → CDN
- [ ] Image Optimizer `quality` 기본값: **90** / `ttype`: 출력 포맷
- [ ] Image Optimizer `anilimit` 기본값: **1** / `autorotate` 기본값: **false**
- [ ] Image Optimizer 변환 옵션: type=f(리사이즈+크롭), type=m(리사이즈), type=wm(워터마크)
- [ ] Video Player Enhancement: HTML5 표준, SaaS, CMAF LL-HLS, Multi DRM
- [ ] Video Player Enhancement 대역폭: **1, 2, 5Mbps**
- [ ] Media Connect Center: Object Storage + VOD Station + Live Station + CDN **통합 관리**
- [ ] Media AI Understanding: 멀티모달(비전+음성 AI), 인물/객체/행동 분석, 자동 자막

---

## 암기 핵심 문장

> "**Live Station: RTMP 입력, HLS 출력, 1080p@60fps, 채널당 5개, 20Mbps, 동시 1만 명**"
> "**VOD Station: nginx-vod-module, Multi DRM, 썸네일 10개, 다중 출력 5개**"
> "**Image Optimizer = Object Storage 원본 → 쿼리스트링으로 실시간 변환 → CDN**"
> "**Media Connect Center = Object Storage + VOD + Live Station + CDN 한 화면 통합 관리**"

---

*다음 챕터: `19_데이터베이스_MySQL_MSSQL.md` → 클라우드 데이터베이스 서비스*
