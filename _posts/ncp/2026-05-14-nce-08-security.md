---
title: "[NCE] 08. 보안"
date: 2026-05-14 10:10:00 +0900
categories: [NCP, NCE]
tags: [NCP, NCE, ACG, NACL, WAF, IDS, IPS, KMS, Certificate Manager, Security Checker, 보안]
description: "ACG/NACL 비교, Web/System/WAS Security Checker 취약점 항목, Certificate Manager, Private CA, KMS, Security Monitoring 정리"
---

## 핵심 요약

- ACG: Stateful, 허용 규칙만, VPC당 500개/NIC당 3개 / NACL: Stateless, 허용+차단, Subnet 전체 적용
- Web Security Checker: SQL Injection, XSS, SSRF, XXE, LFI, RFI, Command Injection 등
- System/WAS Security Checker: Agent 설치 필요, 3가지 점검 가이드(주요정보통신기반시설/전자금융/CSAP)
- Certificate Manager: 만료 1개월 전 알림, 자동 갱신, NAVER Cloud Trust Services 발급
- Private CA: 계정당 최대 10개 CA, 30,000개 인증서
- KMS: 전자 봉투(Envelope Encryption) 방식
- Security Monitoring: Basic(IDS/Anti-Virus) vs Managed(+Anti-DDoS/WAF/IPS)

## 보안 상품 구성

```
Anti-DDoS (고객별 dedicate)
    → Load Balancer
        → IDS/IPS/WAF (Managed Security)
            → ACG (서버 방화벽)
                → Servers
                    → Security Checker (취약점 진단)
```

## ACG vs NACL

| 항목 | Network ACL | ACG |
|------|-------------|-----|
| 적용 단위 | **Subnet** | **서버(NIC)** |
| 규칙 | 허용/차단 모두 | **허용 규칙만** |
| 기본 규칙 | **허용** | **차단** |
| 상태 저장 | **Stateless** | **Stateful** |
| 규칙 평가 | **우선순위** 순서 | 모든 규칙 평가 |
| 적용 방식 | Subnet 내 **모든 서버** 자동 | 서버 시작 시 **지정 필요** |

**ACG 제한**: VPC당 500개, NIC당 3개, Rule In/Out 각 50개

## 방화벽/IDS/IPS/WAF 역할

| 솔루션 | 차단 범위 |
|--------|----------|
| **방화벽** | 네트워크 레이어 - 포트/IP 제한 |
| **IDS** | 서버 OS/소프트웨어 - 불법 통신 **탐지** |
| **IPS** | 서버 OS/소프트웨어 - 위협 탐지 시 **자동 차단** |
| **WAF** | 웹 애플리케이션 레이어 - HTTP 요청 분석, SQL Injection/XSS 탐지 |

## Web Security Checker

고객 웹 사이트의 보안 취약점 진단 서비스.

### 주요 취약점 항목

| 취약점 | 설명 |
|--------|------|
| **SQL Injection** | SQL 구문에 임의 코드 주입 → DB 유출/변조 |
| **XSS** | 악성 스크립트 삽입 → 쿠키/세션 탈취 |
| **SSRF** | 서버가 내부 다른 서버로 의도치 않은 요청 수행 |
| **XXE** | XML External Entity를 악용 → 로컬 파일 열람, DoS |
| **LFI** | 로컬 악성 파일 Include → 임의 명령 실행 |
| **RFI** | 원격 악성 파일 Include → 임의 명령 실행 |
| **Command Injection** | 서버에 직접 명령 전달/실행 |
| **File Upload** | 악성 스크립트 파일 업로드 후 웹 서버 제어 |
| **File Download** | 의도치 않은 파일 다운로드 → 중요 파일 획득 |
| **Directory Listing** | 디렉터리 인덱싱 활성화 → 파일 목록 노출 |
| **Source Code Disclosure** | 스크립트 파일 소스코드 그대로 노출 |
| **Insecure SSL/TLS** | 취약한 SSL/TLS 버전 → MITM 공격 가능 |
| **Mixed Content** | HTTP/HTTPS 혼용 → 비암호화 콘텐츠 수집/변조 |

## System Security Checker (OS Checker)

- Linux/Windows OS 설정 취약점 점검
- **Agent 설치** 필요
- **3가지 점검 가이드**: 주요정보통신기반시설 / 전자금융기반시설 / CSAP

### 주요 Linux 점검 항목

- root 계정 원격 접속 제한 (`PermitRootLogin no`)
- 패스워드 최소 길이: **8자리 이상**
- UMASK: **027 또는 022 권장**
- 로그인 불필요 계정: `/bin/false` Shell 부여
- `/etc/shadow` 퍼미션: **400** (root 읽기만)
- `/etc/passwd` 퍼미션: **644**

## WAS Security Checker

- Apache/Tomcat/Nginx 설정 취약점 점검
- **Agent 설치** 필요, 3가지 점검 가이드 동일

### Apache 주요 점검

- `webdav_modules`, `status_module`, `autoindex_module` 사용 금지
- HTTP TRACE 비활성화 (`TraceEnable off`)
- `ServerTokens Prod`, `ServerSignature off`
- `KeepAlive On`, `MaxKeepAliveRequests 100 이상`, `Timeout 10초 이하`

### Tomcat 주요 점검

- Shutdown port(8005) 사용 금지
- X-Powered-By HTTP Header 비활성화
- autoDeploy/deployOnStartup 사용 금지
- `sslProtocol=TLS` 사용
- 안전하지 않은 Realm(MemoryRealm 등) 사용 제한

### Nginx 주요 점검

- `server_tokens off` (버전 정보 삭제)
- SSL: **SSLv2, SSLv3 금지, TLS 사용**
- `X-XSS-Protection` 설정
- `Strict-Transport-Security` 헤더 추가 (SSL Stripping 방지)
- `nosniff` 옵션 (MIME content-type 스니핑 차단)

## Certificate Manager

- 연계 상품(LB, Image Optimizer)에서 사용할 인증서 등록
- **만료 예정일 한 달 전**부터 알람 메일/SMS 발송
- **인증서 자동 갱신**: DNS 검증 방식으로 발급한 인증서 자동 갱신
- 인증서 발급: **NAVER Cloud Trust Services**

## Private CA

- 내부 서버 간 통신용 사설 인증서 발급/관리
- SSL/TLS: **X.509 표준** 준수
- 계정당 최대 **10개 CA**, **30,000개 인증서**
- CA 키 타입: RSA 2048, RSA 4096, EC 256, EC 521
- Private CA 외부로 **Export 불가**, 외부 키 Import 불가

## KMS (Key Management Service)

**전자 봉투(Envelope Encryption) 방식:**
```
루트 키 → (암호화) → 마스터 키 → (암호화) → 데이터 키 → (암호화) → 데이터
```
- KMS 마스터 키 값은 직접 확인 불가
- 내부 관리자라도 CA 비밀 키 접근 철저히 제어

## Security Monitoring

| 서비스 | Basic | Managed |
|--------|-------|---------|
| **IDS** | 기본 탐지 Rule 적용 | + 고객별 Rule, 주간/월간 보고서 |
| **Anti-DDoS** | **미제공** | 공격 자동 탐지/차단, Public IP/LB 사용자만 |
| **WAF** | **미제공** | 차단 보고서, 주간/월간 보고서 |
| **Anti-Virus** | Windows 기본 제공 | + 탐지 보고서, 예외 처리 |
| **IPS** | **미제공** | 주간/월간 보고서 |

## 추가 보안 상품

| 상품 | 설명 |
|------|------|
| **WebShell Behavior Detector** | 행위 기반 웹쉘 실시간 탐지, Agent 기반 |
| **Cloud Security Watcher** | 멀티 클라우드(AWS/Azure) 통합 보안 모니터링, Agent 기반 |
| **AppSafer** | 모바일 앱 보안 (난독화, 루팅 탐지, 변조 탐지) |
| **FileSafer** | 파일 악성코드 탐지 (HashFilter/FileFilter, REST API) |

## 시험 암기 포인트

- **ACG**: VPC당 500개, NIC당 3개, Stateful, 허용 규칙만
- **NACL**: Stateless, 우선순위 처리, Subnet 전체 자동 적용
- **WAF**: 방화벽/IDS 탐지 불가한 애플리케이션 레이어 공격 탐지
- **Web Security Checker 취약점**: SQL Injection / XSS / SSRF / XXE / LFI / RFI / Command Injection 등
- **System/WAS Security Checker**: Agent 설치 필요, 3가지 점검 가이드
- **Certificate Manager**: 만료 1개월 전 알림, NAVER Cloud Trust Services
- **Private CA**: 최대 10개 CA, 30,000개 인증서, Export 불가
- **KMS**: 전자 봉투 방식 (루트 키 → 마스터 키 → 데이터 키 → 데이터)
- **Security Monitoring Basic**: IDS + Anti-Virus / **Managed**: + Anti-DDoS + WAF + IPS

