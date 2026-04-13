---
title: "NCP Global Edge 504 Timeout, 정말 CDN 기본 3초 제한일까?"
date: 2026-04-03 17:42:00 +0900
categories: [NCP, Troubleshooting]
tags: [ncp, global-edge, cdn, alb, object-storage, timeout, "504", troubleshooting]
description: "Global Edge에서 관측된 504와 3000ms 패턴을 재현 테스트와 CDN/ALB 로그 대조로 검증한 기록"
---

고객 문의 중 이런 유형이 자주 나온다.

- Global Edge 경유 요청에서 간헐적으로 `504 Gateway Timeout`
- `duration` 이 약 `3000ms` 전후에서 반복
- 정적 파일 `MISS` 와 API 요청에서 함께 발생

이 패턴을 보면 가장 먼저 떠오르는 가설은 하나다.

> "Global Edge 기본 origin timeout 이 3초인 것 아닌가?"

이번 글에서는 이 가설을 직접 재현 테스트로 검증한 과정을 정리한다. 결론부터 말하면, **이번 테스트 기준으로는 Global Edge의 고정 3초 origin timeout 을 확인하지 못했다.** 오히려 CDN은 `10초` 지연 응답까지 정상적으로 기다렸다.

추가로 테스트 이후 네이버 클라우드 측 공식 답변도 받았다.

- `timeout`: 3초
- `readTimeout`: 600초
- `firstByteTimeout`: 600초

이 답변을 기준으로 보면, 이번 테스트 결과와 공식 정책은 모순이 아니다. 우리가 재현한 것은 **연결 성공 이후 first byte 를 늦게 보내는 상황**이었고, 이 경우 `firstByteTimeout 600초` 범위 안에서 정상 동작하는 것이 맞다. 반대로 NCP가 말한 `timeout 3초`는 **오리진과의 연결 단계 timeout** 으로 해석하는 것이 자연스럽다.

---

## 문제 정의

고객이 관측한 현상은 다음과 같았다.

- `GET /` 요청에서 `duration ≈ 3000ms`
- `/_next/static/...` 의 `MISS -> 504`
- `/api/v1/mobile/contents/banners/list` 요청에서도 `duration ≈ 3000ms`

정적 파일과 API가 함께 실패했다는 점이 중요했다. 특정 파일 포맷 문제라기보다, **오리진 fetch 단계에서 공통 병목이 있었을 가능성**이 더 높아 보였다.

이번 검증의 목표는 두 가지였다.

1. Global Edge에 기본 `3초` timeout 이 존재하는지 확인
2. `504 @ 3000ms` 패턴이 CDN 문제인지, 원본 계층 문제인지 구분하는 기준 정리
3. 네이버 클라우드 공식 답변의 `timeout 3초 / readTimeout 600초 / firstByteTimeout 600초` 와 재현 테스트 결과를 함께 해석

---

## 테스트 환경

검증에는 아래 환경을 사용했다.

- CDN 도메인: `0lixghcg14234.edge.naverncp.com`
- 정적 사이트 도메인: `fttgdi0c14233.edge.naverncp.com`
- ALB 도메인: `test-alb-target-133396553-bb9d69b9bf3a.kr.lb.naverncp.com`
- ALB Target Group: `lch-cdn-18080`
- Target Type: `VPC Server`
- Health Check: `HTTP / 18080 /healthz`
- 테스트 오리진 서버: 현재 서버의 `18080` 포트

핵심은 간단했다. **같은 요청을 ALB 직접 호출과 CDN 경유 호출로 나눠 비교**했다. 그러면 CDN 구간 문제인지, 원본 계층 문제인지 분리가 가능하다.

---

## 오리진을 어떻게 재현했는가

기존 서비스에 영향을 주지 않기 위해 `18080` 포트에 테스트 전용 HTTP 서버를 별도로 띄웠다.

사용한 파일:

- `/tmp/cdn-origin-test/origin_server.py`
- `/tmp/cdn-origin-test/run-origin-test-server.sh`
- `/tmp/cdn-origin-test/measure_url.py`

이 테스트 서버는 아래 엔드포인트를 제공했다.

- `/healthz`: 즉시 `200`
- `/fast`: 즉시 `200`
- `/headers`: 요청 헤더를 JSON으로 반환
- `/image.png`, `/assets/test.png`, `/images/pixel.png`: 1x1 PNG 반환
- `/delay/1`, `/delay/2`, `/delay/3`, `/delay/4`, `/delay/10`: 지정한 초만큼 지연 후 `200`
- `/delay/4/run-a` 처럼 뒤에 임의 경로를 더 붙여도 동일하게 동작

### 왜 별도 포트와 별도 서버를 썼는가

기존 운영 서비스의 `80` 포트나 `nginx` 설정을 건드리면 테스트 결과가 기존 구성과 섞이기 쉽다. 그래서 아래 원칙으로 분리했다.

- 기존 운영 경로는 그대로 둔다
- 테스트용 응답은 `18080` 포트에서만 제공한다
- ALB Target Group 이 이 테스트 포트를 바라보게 한다
- Health Check 는 `/healthz` 로 단순화한다

이렇게 하면 "현재 운영 서버가 느린 것인지", "테스트 오리진이 일부러 늦게 응답한 것인지"를 명확히 분리할 수 있다.

### 테스트 서버가 실제로 한 일

구현 방식은 매우 단순하다.

1. 요청 경로에서 지연 초를 파싱
2. `sleep(n)` 수행
3. 이후 `200` 응답
4. 응답 헤더 `X-Origin-Test-Elapsed-Ms` 에 실제 지연 시간 기록
5. 오리진 로컬 로그에도 요청 시각과 처리 시간 저장

이 방식의 장점은 분명하다. **오리진이 first byte 를 늦게 보내는 상황**을 명확하게 만들 수 있다.

예를 들어 `/delay/4/run-a` 요청이 들어오면 서버는 아래 순서로 동작한다.

1. 경로에서 `4` 를 읽는다
2. `sleep(4)` 수행
3. 4초가 지난 뒤 JSON 응답을 보낸다
4. 응답 헤더에 `X-Origin-Test-Elapsed-Ms: 4000` 비슷한 값을 넣는다
5. 로컬 로그에 요청 시각, 경로, 실제 처리 시간을 남긴다

즉 이 테스트는 "애플리케이션이 4초 동안 첫 바이트를 보내지 않는 상황"을 의도적으로 만든 것이다.

여기서 중요한 한계도 있다. 이 방식은 **TCP 연결 수립 이후** 를 검증한다. 즉 아래 상황은 검증하지 않는다.

- 오리진 IP로 TCP 연결이 아예 안 붙는 경우
- 특정 포트로 SYN/SYN-ACK 단계가 지연되는 경우
- TLS 핸드셰이크 단계에서 3초 이상 걸리는 경우

따라서 이 테스트로 검증한 것은 정확히 말해:

- `connect timeout 3초` 여부: 직접 검증 아님
- `first byte/read timeout`: 직접 검증

### 왜 경로를 `/delay/4/run-a` 형태로 만들었는가

처음에는 `?ts=` 쿼리 문자열만 붙여서 매번 다른 요청처럼 보이게 하려고 했다. 하지만 Global Edge 기본 캐시 동작에서는 쿼리 문자열이 캐시 키에서 무시될 수 있다.

이 경우 이런 문제가 생긴다.

- `/delay/4?ts=1`
- `/delay/4?ts=2`

두 요청이 사용자 눈에는 다르지만, CDN 캐시 키 관점에서는 같은 객체로 취급될 수 있다. 그러면 첫 요청만 `MISS`, 그 뒤는 `HIT` 이 되어 timeout 재현 테스트가 무의미해진다.

그래서 경로 자체를 바꿨다.

- `/delay/4/run-a`
- `/delay/4/run-b`
- `/delay/10/log-e`

이렇게 하면 CDN 입장에서는 완전히 다른 오브젝트 키로 보게 된다. 즉, **캐시를 피하기 위한 핵심은 쿼리 문자열보다 경로 고유화**였다.

### 이미지 응답도 함께 만든 이유

실제 고객 문의에서는 정적 파일(`woff2`, `css`, `js`)과 API가 함께 실패했다. 그래서 JSON 응답뿐 아니라 이미지 응답도 추가했다.

- `/image.png`
- `/assets/test.png`
- `/images/pixel.png`

이 경로들은 1x1 PNG 파일을 반환하도록 구성했다. 목적은 단순하다.

- 정적 에셋 계열 요청도 CDN 로그에 남게 하기
- `Content-Type: image/png` 응답을 분리해서 보기
- API와 에셋이 같은 방식으로 실패하는지 비교하기

### 오리진 로컬 로그는 왜 필요했는가

CDN 로그와 ALB 로그만 보면 "CDN이 몇 ms 동안 기다렸는지"는 볼 수 있어도, 오리진이 실제로 몇 ms 지연했는지는 확정하기 어렵다.

그래서 오리진 서버도 별도 로그를 남기게 했다.

로컬 로그에는 아래 정보가 기록됐다.

- 요청 시각
- 클라이언트 IP
- 요청 경로
- HTTP 상태
- 실제 처리 시간
- 설정된 지연 초

즉 세 개 로그를 서로 맞출 수 있었다.

- 오리진 로컬 로그: 실제 지연이 몇 초였는지
- ALB 로그: ALB가 몇 초 동안 응답을 기다렸는지
- CDN 로그: Global Edge가 몇 초 동안 응답을 기다렸는지

이 세 개가 맞아떨어지면 "어느 구간에서 timeout 이 났는지"를 훨씬 명확하게 판단할 수 있다.

### 실제 실행 방식

테스트 서버는 아래처럼 실행했다.

```bash
ORIGIN_TEST_PORT=18080 /tmp/cdn-origin-test/run-origin-test-server.sh
```

ALB는 아래처럼 붙였다.

- Target Group 포트: `18080`
- Health Check: `HTTP /healthz`

헬스체크가 `UP` 으로 바뀐 뒤부터 ALB 직접 호출과 CDN 경유 호출을 시작했다.

### 측정 스크립트는 무엇을 기록했는가

`measure_url.py` 는 아래 값을 출력했다.

- HTTP 상태코드
- `ttfb_ms`
- `total_ms`
- 응답 바이트 길이
- 오리진 헤더 `X-Origin-Test-Elapsed-Ms`
- 요청 URL

예를 들어 `/delay/10/run-a` 를 CDN 경유로 호출했을 때:

- `status=200`
- `ttfb_ms ≈ 10031`
- `origin_elapsed_ms=10000`

이렇게 나오면 "CDN이 10초짜리 오리진 응답을 실제로 기다렸다"는 뜻이다.

---

## 테스트 전략

### 1. ALB 직접 호출

먼저 ALB 도메인으로 직접 호출했다. 목적은 원본 계층이 실제로 몇 초까지 버티는지 확인하는 것이다.

```bash
python3 /tmp/cdn-origin-test/measure_url.py \
  --url "http://test-alb-target-133396553-bb9d69b9bf3a.kr.lb.naverncp.com/delay/10/run-a" \
  --repeat 1 \
  --timeout 20
```

### 2. CDN 경유 호출

같은 경로를 Global Edge 도메인으로 다시 호출했다.

```bash
python3 /tmp/cdn-origin-test/measure_url.py \
  --url "http://0lixghcg14234.edge.naverncp.com/delay/10/run-a" \
  --repeat 1 \
  --timeout 20
```

### 3. 캐시 영향 제거

Global Edge 기본 캐시 설정은 쿼리 문자열을 캐시 키에서 무시할 수 있다. 그래서 `?ts=` 만 붙이면 완전한 `MISS` 강제가 안 될 수 있다.

이 문제를 피하려고 두 가지를 함께 사용했다.

- 쿼리 문자열 추가
- 경로 자체를 고유하게 변경
  - `/delay/4/run-a`
  - `/delay/4/run-b`

즉, **캐시 키 수준에서 완전히 다른 오브젝트처럼 보이게 만들었다.**

### 4. 로그 대조

테스트는 응답만 보고 끝내지 않았다. 아래 로그를 함께 봤다.

- Global Edge 표준 로그
  - `sc-cachehit`
  - `sc-status`
  - `time-response`
  - `time-taken`
  - `sc-request-id`
- ALB 액세스 로그
  - 요청 시각
  - 응답 코드
  - 처리 시간
- 오리진 로컬 로그
  - 요청 경로
  - 실제 지연 시간

---

## 실제 관측 결과

### ALB 직접 호출

ALB 직접 호출은 아래처럼 나왔다.

- `/delay/1` -> `200`, 약 `1000ms`
- `/delay/2` -> `200`, 약 `2000ms`
- `/delay/3` -> `200`, 약 `3000ms`
- `/delay/4` -> `200`, 약 `4000ms`
- `/delay/10/run-a` -> `200`, 약 `10000ms`

즉 원본 계층은 최소 `10초` 지연 응답까지 정상 처리했다.

### CDN 경유 호출

Global Edge 경유 결과도 거의 동일했다.

- `/delay/1/log-a` -> `MISS 200`, `1009ms`
- `/delay/2/log-b` -> `MISS 200`, `2005ms`
- `/delay/3/log-c` -> `MISS 200`, `3009ms`
- `/delay/4/log-d` -> `MISS 200`, `4008ms`
- `/delay/10/log-e` -> `MISS 200`, `10004ms`

여기서 가장 중요한 건 `/delay/10/log-e` 였다. **CDN이 오리진의 `10초` 지연 응답을 정상적으로 기다린 뒤 `200`으로 반환했다.**

이 결과는 네이버 클라우드 공식 답변의 아래 내용과 잘 들어맞는다.

- `timeout`: 3초
- `readTimeout`: 600초
- `firstByteTimeout`: 600초

즉 이번 테스트는 `firstByteTimeout` 과 `readTimeout` 범위 안에서 정상 동작한 사례로 해석할 수 있다.

---

## CDN 로그가 말해준 것

실제 수집된 Global Edge 표준 로그 일부는 다음과 같았다.

- `/delay/1/log-a` -> `MISS 200`, `time-response=1009`, `time-taken=1009`
- `/delay/2/log-b` -> `MISS 200`, `time-response=2005`, `time-taken=2005`
- `/delay/3/log-c` -> `MISS 200`, `time-response=3009`, `time-taken=3009`
- `/delay/4/log-d` -> `MISS 200`, `time-response=4008`, `time-taken=4008`
- `/delay/10/log-e` -> `MISS 200`, `time-response=10004`, `time-taken=10004`

이 로그는 아주 직접적이다.

- CDN은 `3초`에서 끊기지 않았다
- `4초`에서도 끊기지 않았다
- `10초`에서도 끊기지 않았다

즉, **이번 구성에서 `504 @ 3000ms` 패턴은 재현되지 않았다.**

정확히 말하면 이번 테스트는 "Global Edge가 first byte 를 3초만 기다리는가"를 확인한 것이고, 결과는 **아니었다** 이다.

---

## ALB 로그가 말해준 것

ALB 액세스 로그도 CDN 경유 요청이 정상적으로 ALB까지 내려왔음을 보여줬다.

예를 들어:

- `0lixghcg14234.edge.naverncp.com/delay/3?...` -> `200`, `3001`
- `0lixghcg14234.edge.naverncp.com/delay/4?...` -> `200`, `4001`
- `0lixghcg14234.edge.naverncp.com/delay/10/run-a` -> `200`, `10001`

즉,

- CDN이 요청을 ALB까지 정상 전달했고
- ALB는 오리진의 지연 응답을 정상 전달했다

따라서 이번 테스트에서는 CDN과 ALB 사이, ALB와 오리진 사이 모두 timeout 문제가 없었다.

또한 ALB 로그에서 `10초` 지연 요청도 정상 `200` 으로 처리된 것을 보면, 적어도 이 재현 환경에서는 ALB가 3초에서 요청을 끊지도 않았다.

---

## 네이버 클라우드 답변과 테스트 결과는 왜 다르게 보였나

테스트 후 네이버 클라우드로부터 아래와 같은 답변을 받았다.

- `timeout`: 3초
- `readTimeout`: 600초
- `firstByteTimeout`: 600초
- 여기서 `timeout 3초` 는 원본 서버와의 통신 시 timeout

처음 보면 이 답변은 우리가 본 `10초 정상 응답` 과 충돌하는 것처럼 보인다. 하지만 실제로는 그렇지 않다.

### 우리가 테스트한 것

우리가 만든 `/delay/10/run-a` 테스트는 아래 상황을 재현했다.

1. CDN이 ALB와 정상적으로 연결한다
2. ALB도 오리진과 정상적으로 연결한다
3. 오리진 애플리케이션이 first byte 를 `10초` 늦게 보낸다
4. 이후 응답은 정상 `200` 으로 완료된다

즉, 이 테스트는 **"연결이 된 뒤, 응답 시작이 늦은 상황"** 을 본 것이다.

### NCP가 말한 `timeout 3초`가 의미하는 것

공식 답변 문맥상 `timeout 3초`는 아래 범주로 보는 것이 타당하다.

- 오리진과 TCP 연결을 맺는 단계
- 혹은 오리진과 프로토콜별 정상 통신을 시작하는 초기 연결 단계

즉, 연결 자체가 늦거나 막히는 경우라면 `3초`에서 실패할 수 있다.

예를 들어 이런 경우다.

- 오리진 IP/포트가 닫혀 있음
- 방화벽/ACG 문제로 SYN 단계가 지연됨
- TLS 핸드셰이크가 비정상적으로 오래 걸림
- 특정 프로토콜(HTTP/HTTPS) 설정이 맞지 않아 초반 연결이 오래 걸림

### 그래서 고객 로그의 3000ms는 무엇을 의미할 수 있나

고객이 본 `504 @ 약 3000ms` 는 이제 두 갈래로 해석할 수 있다.

1. 실제로는 연결 단계에서 3초 안에 정상 통신을 시작하지 못했다
2. 또는 고객 환경의 특정 경로/시점에서 ALB/backend 가 초기 응답을 제때 시작하지 못했다

즉, "Global Edge는 무조건 3초 후 504를 낸다" 가 아니라, **특정 환경에서 오리진과의 초기 통신이 3초 내에 성립하지 못한 것일 수 있다.**

---

## 결론

이번 테스트 기준 결론은 명확하다.

1. 공개 문서만으로는 Global Edge의 기본 `connect/response/first-byte timeout` 값을 확인하지 못했다.
2. 이후 네이버 클라우드 공식 답변으로 `timeout 3초 / readTimeout 600초 / firstByteTimeout 600초` 정책을 확인했다.
3. 실제 재현 테스트 결과, Global Edge는 최소 `10초`의 오리진 지연 응답을 정상 처리했다.
4. 따라서 고객 환경에서 관측된 `504 @ 약 3000ms` 패턴을 "Global Edge가 first byte 를 3초만 기다린 결과" 로 해석하면 안 된다.
5. 오히려 고객 사례는 오리진과의 **초기 연결 또는 프로토콜별 통신 시작 단계** 문제로 다시 보는 것이 더 자연스럽다.

즉, 고객이 본 `3000ms` 패턴은 **Global Edge의 보편적인 기본 제한값이라기보다, 특정 요청, 특정 환경, 특정 시점에서 발생한 현상**으로 보는 것이 맞다.

---

## 그럼 504는 어떻게 판별해야 할까

실무에서는 아래처럼 보면 된다.

### 경우 1. CDN 로그 `504` + ALB 로그도 같은 시점 `504`

이 경우는 원본 계층 문제 가능성이 높다.

- 애플리케이션 지연
- 백엔드 upstream 지연
- ALB 뒤 서버 연결 실패

### 경우 2. CDN 로그 `504` + ALB 로그는 `200`

이 경우는 CDN-origin fetch 구간 또는 CDN 측 조건 문제 가능성이 있다.

- 특정 `MISS` 조건
- 특정 Host/Header 조건
- 특정 경로에서만 발생하는 fetch 이슈

### 경우 3. CDN 로그 `504` + ALB 로그에 요청 자체가 없음

이 경우는 CDN에서 ALB 도달 전 문제일 수 있다.

- 연결 실패
- 보안그룹/ACG
- 리스너/라우팅 문제

핵심은 단순하다.

> `504 @ 3000ms` 패턴만으로 CDN 기본 timeout 문제라고 단정하면 안 된다. 반드시 CDN 로그와 ALB 로그를 같이 봐야 한다.

---

## 정적 사이트를 Object Storage 오리진으로 쓸 때 주의할 점

추가로 정적 사이트 테스트 중 하나 더 확인한 점이 있다.

Object Storage를 Global Edge 오리진으로 쓸 때 `오리진 경로`에 `/index.html` 을 넣으면 안 된다.

이유는 간단하다.

- `/assets/app.js` 요청이 `/index.html/assets/app.js` 로 변형될 수 있기 때문이다.

정적 사이트는 아래처럼 구성하는 게 맞다.

- 오리진: `NCP Object Storage`
- Forward Host Header: `Origin Hostname`
- 오리진 프로토콜: `HTTPS 443`
- 오리진 경로: 비움
- 별도 Rule Builder 에서 `/` 요청만 `/index.html` 로 `URL Rewrite`

즉, 루트 페이지 문제는 `오리진 경로`가 아니라 `URL Rewrite` 로 해결해야 한다.

---

## 후속 권장사항

고객 환경에서 실제 원인을 좁히려면 아래 순서가 가장 효율적이다.

1. 고객이 실제로 504를 본 URI 그대로 다시 테스트
2. 같은 시각의 CDN 로그와 ALB 로그를 매칭
3. 가능하면 앱 로그까지 함께 확인
4. `MISS -> 504` 가 난 실제 요청의 `sc-request-id` 기준으로 역추적

이번 검증의 핵심 메시지는 이 한 줄로 정리된다.

> 이번 재현 테스트는 `first byte/read timeout` 관점에서는 정상 통과했다. 고객 환경의 `504 @ 3000ms` 는 `connect timeout 3초` 관점에서 다시 봐야 한다.
