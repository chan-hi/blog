---
title: "NCP Archive Storage 접속 자동화 개발기"
date: 2026-04-28 10:00:00 +0900
categories: [NCP, Troubleshooting]
tags: [ncp, archive-storage, rclone, powershell, openstack, keystone, automation, swift]
description: "S3 Browser로 접근 불가한 NCP Archive Storage를 rclone + PowerShell로 자동화하여 바탕화면 클릭 한 번에 브라우저로 탐색하는 도구를 만든 과정"
---

## 개요

대학병원 교수 연구팀의 연구 데이터 보관 비용을 절감하기 위해 NCP Object Storage에서 Archive Storage로 이관한 후, 기존 도구로는 접근이 불가능한 문제를 해결하고 비기술 사용자도 클릭 한 번으로 파일을 탐색할 수 있는 자동화 도구를 개발한 과정을 기록한다.

---

## 배경 및 문제 상황

대학병원 교수 연구팀은 수백 GB 규모의 연구 데이터를 NCP(Naver Cloud Platform) Object Storage에 보관하고 있었으며, S3 Browser라는 GUI 툴로 파일을 관리해왔다. 비용 절감을 위해 Archive Storage로 이관을 완료했으나, 이후 S3 Browser에서 **한글 폴더명이 전혀 표시되지 않는다**는 문제가 발생했다.

연구 데이터 특성상 한글 폴더명이 다수 사용되고 있었기 때문에, 실질적으로 데이터에 접근하는 것이 불가능한 상황이었다.

---

## 기술적 원인 분석

문제의 핵심은 **스토리지 프로토콜의 차이**였다.

| 서비스 | 프로토콜 | 인증 방식 |
|--------|---------|---------|
| NCP Object Storage | S3 호환 API | AWS Signature |
| NCP Archive Storage | OpenStack Swift v3 | Keystone v3 토큰 |

NCP Object Storage는 AWS S3와 호환되는 API를 제공하여 S3 Browser, Cyberduck 등 범용 S3 클라이언트로 접근이 가능하다. 반면 Archive Storage는 **OpenStack Swift v3 방식으로만 접근**할 수 있다.

S3 Browser로 Archive Storage에 S3 엔드포인트를 통해 접속하면 연결 자체는 되지만, OpenStack Swift가 내부적으로 사용하는 한글 인코딩 방식과 S3 API의 인코딩 처리 방식이 달라 한글 폴더명이 깨지거나 아예 보이지 않는 문제가 발생한다.

---

## 1차 시도: GUI 클라이언트 연동 (실패)

### 시도 내용

OpenStack Swift v3를 지원하는 GUI 클라이언트를 활용하기로 하고, 가장 널리 쓰이는 **Cyberduck**과 **CloudBerry Explorer**를 시도했다.

두 도구 모두 OpenStack Swift v3 연결 옵션을 제공하며, 설정 항목은 다음과 같다.

- Auth URL: `https://kr.archive.ncloudstorage.com:5000/v3`
- Username: `ncp_iam_XXXXXXXXXXXXXXXXXXXX`
- Password: (IAM Secret Key)
- Project ID: `<PROJECT_ID>`

### 실패 원인

두 도구 모두 **401 Unauthorized** 오류가 발생했다. 원인을 분석한 결과, OpenStack Keystone v3 인증 시 도메인을 지정하는 방식의 차이가 문제였다.

OpenStack 표준 스펙에서는 도메인을 **이름(name)** 또는 **ID** 두 가지 방식으로 전달할 수 있다.

```json
// 이름 방식 (일반 OpenStack — 대부분의 GUI 클라이언트가 사용)
"domain": { "name": "default" }

// ID 방식 (NCP Archive Storage가 요구하는 방식)
"domain": { "id": "default" }
```

Cyberduck과 CloudBerry는 내부적으로 이름(name) 방식으로 인증 요청을 전송하도록 하드코딩되어 있었고, NCP Archive Storage는 **ID 방식만 허용**하기 때문에 모든 GUI 클라이언트에서 인증 실패가 발생했다. 설정 화면에서 Domain ID를 입력하는 필드가 있더라도, 실제 HTTP 요청에서는 name 방식으로 전송되는 경우가 대부분이었다.

---

## 2차 시도: rclone + Keystone 토큰 직접 주입 (성공)

### 접근 방식 전환

GUI 클라이언트가 인증 방식을 수정할 수 없는 구조라면, **인증 단계를 우회**하는 방법을 찾기로 했다. OpenStack Keystone 서버에 직접 토큰을 발급받아 rclone에 주입하면, rclone이 재인증 없이 해당 토큰을 그대로 사용하게 된다.

### 인증 흐름

```
[기존 실패 방식]
rclone → Keystone 서버 (name 방식 인증 시도) → 401 Unauthorized

[해결 방식]
PowerShell → Keystone 서버 (ID 방식 인증) → 토큰 발급
토큰 → rclone.conf에 직접 저장
rclone → Archive Storage (저장된 토큰 사용) → 인증 성공
```

### Keystone 토큰 발급 (PowerShell)

```powershell
$body = @{
    auth = @{
        identity = @{
            methods  = @("password")
            password = @{
                user = @{
                    name     = $USERNAME
                    password = $PASSWORD
                    domain   = @{ id = "default" }   # name이 아닌 id 방식
                }
            }
        }
        scope = @{
            project = @{ id = $PROJECT_ID }
        }
    }
} | ConvertTo-Json -Depth 10

$response = Invoke-WebRequest `
    -Uri "https://kr.archive.ncloudstorage.com:5000/v3/auth/tokens" `
    -Method POST `
    -ContentType "application/json" `
    -Body $body `
    -UseBasicParsing

$TOKEN = $response.Headers["X-Subject-Token"]
```

핵심은 `domain = @{ id = "default" }`로, 이름(name)이 아닌 **ID 방식을 명시적으로 사용**하는 것이다.

### rclone 설정 (토큰 주입 방식)

```ini
[ncp-archive]
type = swift
env_auth = false
storage_url = https://kr.archive.ncloudstorage.com/v1/AUTH_<PROJECT_ID>
auth_token = <발급된 Keystone 토큰>
```

rclone의 일반 Swift 연결 방식은 내부적으로 Keystone 재인증을 시도하기 때문에 동일한 문제가 발생한다. `storage_url` + `auth_token` 조합을 사용하면 rclone이 Keystone 서버에 접근하지 않고 토큰을 그대로 사용하므로 인증 문제를 완전히 우회할 수 있다.

> 토큰은 발급 후 **24시간 후 만료**되므로 매일 재발급이 필요하다.

### 브라우저 UI 구현

명령줄에 익숙하지 않은 사용자를 위해 rclone의 HTTP 서버 기능을 활용했다.

```bash
rclone serve http ncp-archive: --addr 127.0.0.1:18080 --no-modtime
```

이 명령어 하나로 로컬에 HTTP 서버가 실행되고, 웹 브라우저에서 `http://127.0.0.1:18080`으로 접속하면 Archive Storage의 파일을 폴더 형태로 탐색하고 다운로드할 수 있게 된다.

### 대용량 파일(5GB 이상) 처리

Archive Storage는 5GB 이상 파일을 자동으로 세그먼트(조각)로 분할 저장한다(DLO/SLO 방식). 브라우저 UI에서도 rclone이 세그먼트를 내부적으로 합쳐 단일 파일로 스트리밍하기 때문에, 사용자는 10GB 파일도 클릭 한 번으로 다운로드할 수 있었다.

업로드 시에는 `--swift-chunk-size 1G` 옵션으로 1GB 단위 자동 분할 업로드를 지원한다.

---

## 3차 시도: 설치 및 실행 자동화 (PowerShell 스크립트)

### 목표

최종 사용자인 교수는 명령줄 도구 사용 경험이 전혀 없다. 매일 사용 시작 시 토큰 발급 → rclone.conf 수정 → 서버 실행의 과정을 수동으로 반복하는 것은 현실적으로 불가능하다. **전체 과정을 자동화하여 바탕화면 아이콘 클릭 한 번으로 브라우저가 열리는 환경**을 목표로 했다.

### 설계 구조

```
[배포 패키지]
NCP_Archive_Install.bat          → 사용자가 더블클릭하는 진입점
NCP_Archive_Install.ps1          → 실제 설치 로직 (PowerShell)
rclone-v1.73.5-windows-amd64.zip → rclone 바이너리 (오프라인 설치 지원)

[설치 후 생성 파일]
C:\NCP_Archive\
  ├── rclone.exe      → rclone 바이너리
  ├── rclone.conf     → 스토리지 연결 설정 (토큰 포함)
  ├── launch.ps1      → 매번 실행 시 토큰 갱신 + 서버 시작
  └── launcher.vbs    → PowerShell을 검은 창 없이 실행하는 래퍼

[바탕화면]
NCP File Browser.lnk → launcher.vbs 바로가기
```

### 배치파일(.bat) + PowerShell(.ps1) 분리 구조

Windows의 기본 실행 정책(ExecutionPolicy)은 `.ps1` 파일의 더블클릭 실행을 차단한다. `.bat` 파일은 이 제한 없이 실행되므로, 사용자에게는 `.bat` 파일을 전달하고 내부에서 `-ExecutionPolicy Bypass` 옵션으로 `.ps1`을 실행하는 방식을 채택했다.

```batch
@echo off
powershell -NoProfile -ExecutionPolicy Bypass -File "%~dp0NCP_Archive_Install.ps1"
```

### rclone 자동 설치 (인터넷/오프라인 양쪽 지원)

```powershell
if (Test-Path $RCLONE_EXE) {
    # 이미 설치됨 → 건너뜀
} elseif (Test-Path $RCLONE_ZIP) {
    # zip 파일이 옆에 있음 → 압축 해제
    Expand-Archive -Path $RCLONE_ZIP -DestinationPath "$env:TEMP\rclone_setup" -Force
} else {
    # 아무것도 없음 → 인터넷에서 다운로드
    Invoke-WebRequest -Uri "https://downloads.rclone.org/rclone-current-windows-amd64.zip" -OutFile $zip
}
```

### VBScript 래퍼를 통한 무창(Silent) 실행

PowerShell 창(검은 터미널)이 잠깐이라도 뜨면 사용자가 놀라거나 혼동할 수 있다. VBScript로 PowerShell을 숨김 모드(`WindowStyle Hidden`)로 실행하는 래퍼를 작성했다.

```vbscript
Set s = CreateObject("Wscript.Shell")
s.Run "powershell.exe -NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -File ""C:\NCP_Archive\launch.ps1""", 0, False
```

이때 **UTF-8 BOM 문제**가 발생했다. PowerShell의 기본 `WriteAllText`는 UTF-8 BOM을 포함하여 파일을 작성하는데, VBScript는 BOM이 있는 파일을 파싱하지 못하고 `800A0408 유효하지 않은 문자입니다` 오류를 발생시킨다. `System.Text.UTF8Encoding $false`를 명시적으로 사용하여 BOM 없이 저장하는 것으로 해결했다.

```powershell
$utf8NoBom = New-Object System.Text.UTF8Encoding $false
[System.IO.File]::WriteAllLines($LAUNCHER_VBS, $vbsLines, $utf8NoBom)
```

### 포트 충돌 방지

동일 PC에 MSP 관리용(포트 18080)과 고객사용(포트 18081) 두 가지 설치본이 공존할 수 있도록 설계했다. 실행 시 해당 포트를 사용 중인 프로세스만 선택적으로 종료한다.

```powershell
# 모든 rclone을 종료하는 대신, 해당 포트만 선택적으로 종료
$conn = Get-NetTCPConnection -LocalPort $PORT -ErrorAction SilentlyContinue | Where-Object State -eq "Listen"
if ($conn) { Stop-Process -Id $conn.OwningProcess -Force -ErrorAction SilentlyContinue }
```

### launch.ps1 실행 흐름

바탕화면 아이콘 클릭 시 `launch.ps1`이 실행되며 다음 과정이 자동으로 진행된다.

```
1. 기존 rclone 서버 종료 (해당 포트만)
2. Keystone 서버에 토큰 발급 요청 (ID 방식)
3. rclone.conf에 새 토큰 저장
4. rclone HTTP 서버 백그라운드 실행
5. 브라우저 자동 오픈 (http://127.0.0.1:18080)
```

---

## 개발 중 해결한 주요 문제들

| 문제 | 원인 | 해결책 |
|------|------|--------|
| 모든 GUI 클라이언트 401 오류 | Keystone v3 인증 시 domain name 방식 사용 | PowerShell로 직접 토큰 발급 후 rclone에 주입 |
| VBScript 800A0408 오류 | UTF-8 BOM이 포함된 파일 생성 | `UTF8Encoding $false`로 BOM 없이 저장 |
| 배치 파일 인코딩 오류 | 한글이 포함된 .bat 파일의 ANSI/UTF-8 충돌 | .bat에서 한글 제거, 로직 전체를 .ps1로 이전 |
| 두 설치본 포트 충돌 | 동일 포트 사용, `Stop-Process -Name rclone`이 모든 인스턴스 종료 | 포트 분리(18080/18081) + 포트 기준 프로세스 선택 종료 |
| 5GB 이상 세그먼트 파일 | DLO/SLO 분할 저장 방식 | rclone이 세그먼트를 자동 병합하여 단일 파일로 제공 |

---

## 결과

- 비기술 사용자가 바탕화면 아이콘 더블클릭 한 번으로 NCP Archive Storage를 브라우저에서 탐색 가능
- 토큰 발급, 설정 파일 갱신, 서버 실행 전 과정 자동화
- 10GB 이상 세그먼트 파일도 브라우저에서 클릭 한 번으로 다운로드 가능
- MSP 관리용과 고객사용 두 가지 배포 패키지로 분리하여 동일 PC에서 공존 가능
- 오프라인 환경(인터넷 차단 PC)에서도 zip 동봉 방식으로 설치 가능

---

## 기술 스택

- **rclone** v1.73.5 — OpenStack Swift v3 스토리지 클라이언트 + HTTP 서버
- **PowerShell** — 토큰 발급 자동화, 설치 스크립트
- **VBScript** — PowerShell 무창(Silent) 실행 래퍼
- **OpenStack Keystone v3** — NCP Archive Storage 인증
- **NCP Archive Storage** — OpenStack Swift v3 기반 오브젝트 스토리지
