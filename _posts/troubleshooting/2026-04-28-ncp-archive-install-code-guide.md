---
title: "NCP_Archive_Install.ps1 코드 해설"
date: 2026-04-28 10:30:00 +0900
categories: [NCP, Troubleshooting]
tags: [ncp, archive-storage, rclone, powershell, openstack, keystone, windows, script]
description: "NCP Archive Storage 자동화 설치 스크립트(PowerShell)의 전체 구조와 각 코드 블록을 상세히 해설한 문서"
---

> 이 포스트는 [NCP Archive Storage 접속 자동화 개발기](/posts/ncp-archive-automation/)의 후속으로, 실제 설치 스크립트 코드를 블록 단위로 상세히 해설한다.

## 전체 구조

이 스크립트는 실행되면 크게 두 가지 일을 한다.

1. **설치 (1회)** — rclone 설치, 스크립트 파일 생성, 바탕화면 단축키 생성
2. **런처 생성** — 이후 매번 실행될 `launch.ps1`을 파일로 써두기

```
NCP_Archive_Install.ps1 실행 (1회)
│
├── rclone.exe 설치
├── launch.ps1 생성      ← 매일 아이콘 클릭 시 실행되는 핵심 스크립트
├── launcher.vbs 생성    ← 검은 창 없이 launch.ps1을 실행하는 래퍼
└── 바탕화면 단축키 생성
```

---

## 1. 상단 초기화

```powershell
Add-Type -AssemblyName System.Windows.Forms
$ErrorActionPreference = "Continue"
```

- `Add-Type -AssemblyName System.Windows.Forms` — Windows GUI 팝업창(`MessageBox`)을 쓰기 위해 .NET 라이브러리를 로드한다. 이게 없으면 `[System.Windows.Forms.MessageBox]::Show(...)` 같은 팝업 호출이 실패한다.
- `$ErrorActionPreference = "Continue"` — 오류가 발생해도 스크립트가 멈추지 않고 계속 진행한다. `"Stop"`으로 설정하면 어떤 오류든 즉시 스크립트가 종료된다. 개발 초기에 이 값 때문에 창이 갑자기 닫히는 문제가 있었다.

---

## 2. 변수 선언

```powershell
$INSTALL_DIR = "C:\NCP_Archive"
$RCLONE_ZIP  = Join-Path $PSScriptRoot "rclone-v1.73.5-windows-amd64.zip"
$RCLONE_EXE  = Join-Path $INSTALL_DIR "rclone.exe"
$LAUNCH_PS1  = Join-Path $INSTALL_DIR "launch.ps1"
$LAUNCHER_VBS= Join-Path $INSTALL_DIR "launcher.vbs"
$DESKTOP     = [System.Environment]::GetFolderPath("Desktop")
$SHORTCUT    = Join-Path $DESKTOP "NCP File Browser.lnk"
```

- `$PSScriptRoot` — 이 스크립트 파일이 있는 폴더 경로다. `Join-Path $PSScriptRoot "파일명"`을 쓰면 스크립트와 같은 폴더에 있는 파일을 참조할 수 있다.
- `[System.Environment]::GetFolderPath("Desktop")` — 현재 사용자의 바탕화면 경로를 가져온다. Windows 사용자마다 경로가 다를 수 있으므로(`C:\Users\사용자명\Desktop`) 하드코딩하지 않고 이 방식을 쓴다.
- `Join-Path` — 경로를 합친다. `"C:\NCP_Archive" + "rclone.exe"` → `"C:\NCP_Archive\rclone.exe"`. 슬래시를 직접 쓰는 것보다 안전하다.

---

## 3. 출력 헬퍼 함수

```powershell
function Show-Step($msg) { Write-Host "  >>  $msg" -ForegroundColor Cyan }
function Show-OK($msg)   { Write-Host "  OK  $msg" -ForegroundColor Green }
function Show-Fail($msg) {
    Write-Host "  !!  $msg" -ForegroundColor Red
    [System.Windows.Forms.MessageBox]::Show($msg, "Install Error", "OK", "Error") | Out-Null
    exit 1
}
```

설치 진행 상황을 콘솔에 색깔로 구분해서 출력하는 함수들이다.

- `Show-Step` — 진행 중 (파란색 `>>`)
- `Show-OK` — 완료 (초록색 `OK`)
- `Show-Fail` — 오류 (빨간색 `!!`) + 팝업창 띄우고 스크립트 강제 종료

`| Out-Null`은 `MessageBox.Show()`의 반환값(클릭한 버튼 이름)을 버리는 용도다. 없으면 콘솔에 `OK` 같은 문자열이 출력된다.

---

## 4. 설치 폴더 생성

```powershell
if (-not (Test-Path $INSTALL_DIR)) {
    New-Item -ItemType Directory -Path $INSTALL_DIR -Force | Out-Null
}
```

- `Test-Path` — 파일 또는 폴더가 존재하는지 확인한다. `-not`을 붙여 "없으면"을 표현한다.
- `New-Item -ItemType Directory` — 폴더를 만든다. `-Force`는 이미 존재해도 오류 없이 넘어간다.
- 이미 설치된 경우 폴더가 있으므로 이 단계는 건너뛴다.

---

## 5. rclone 설치 (3단계 폴백 구조)

```powershell
if (Test-Path $RCLONE_EXE) {
    Show-OK "rclone already installed"

} elseif (Test-Path $RCLONE_ZIP) {
    Expand-Archive -Path $RCLONE_ZIP -DestinationPath "$env:TEMP\rclone_setup" -Force
    $exe = Get-ChildItem "$env:TEMP\rclone_setup" -Recurse -Filter "rclone.exe" | Select-Object -First 1
    if (-not $exe) { Show-Fail "rclone.exe not found in zip." }
    Copy-Item $exe.FullName $RCLONE_EXE -Force

} else {
    $url = "https://downloads.rclone.org/rclone-current-windows-amd64.zip"
    $zip = "$env:TEMP\rclone_dl.zip"
    Invoke-WebRequest -Uri $url -OutFile $zip -UseBasicParsing -ErrorAction Stop
    Expand-Archive -Path $zip -DestinationPath "$env:TEMP\rclone_setup" -Force
    $exe = Get-ChildItem "$env:TEMP\rclone_setup" -Recurse -Filter "rclone.exe" | Select-Object -First 1
    Copy-Item $exe.FullName $RCLONE_EXE -Force
}
```

세 가지 상황을 순서대로 처리한다.

| 순서 | 조건 | 동작 |
|------|------|------|
| 1 | `rclone.exe`가 이미 설치됨 | 건너뜀 |
| 2 | zip 파일이 스크립트 옆에 있음 | zip에서 압축 해제 후 설치 |
| 3 | 아무것도 없음 | 인터넷에서 다운로드 후 설치 |

- `$env:TEMP` — Windows 임시 폴더 경로(`C:\Users\사용자명\AppData\Local\Temp`)다.
- `Expand-Archive` — zip 압축 해제다.
- `Get-ChildItem -Recurse -Filter "rclone.exe"` — 압축 해제된 폴더 안에서 `rclone.exe`를 재귀 검색한다. zip 파일 내부에 버전별 하위 폴더가 있어서 경로가 일정하지 않기 때문에 검색 방식을 쓴다.
- `Select-Object -First 1` — 검색 결과 중 첫 번째만 가져온다.
- `Invoke-WebRequest -UseBasicParsing` — 인터넷에서 파일을 다운로드한다. `-UseBasicParsing`은 Internet Explorer 엔진 없이도 동작하도록 하는 옵션이다.

---

## 6. launch.ps1 생성 (핵심)

```powershell
$launchContent = @'
...
'@
[System.IO.File]::WriteAllText($LAUNCH_PS1, $launchContent, [System.Text.Encoding]::UTF8)
```

`@'...'@`은 PowerShell의 **히어독(Here-String)** 문법이다. 따옴표나 특수문자를 그대로 포함한 여러 줄 문자열을 만들 때 쓴다. 이 안의 내용이 `launch.ps1` 파일로 그대로 저장된다.

`launch.ps1` 내부 동작은 아이콘 클릭 시마다 실행되며 다음 순서로 진행된다.

### 6-1. 기존 rclone 프로세스 종료

```powershell
$conn = Get-NetTCPConnection -LocalPort $PORT -ErrorAction SilentlyContinue | Where-Object State -eq "Listen"
if ($conn) { Stop-Process -Id $conn.OwningProcess -Force -ErrorAction SilentlyContinue }
Start-Sleep -Seconds 1
```

- `Get-NetTCPConnection -LocalPort $PORT` — 해당 포트를 사용 중인 TCP 연결을 조회한다.
- `Where-Object State -eq "Listen"` — 그 중 Listen(대기) 상태인 것만 필터링한다.
- `$conn.OwningProcess` — 해당 포트를 점유한 프로세스 ID다.
- `Stop-Process -Id` — 해당 프로세스만 종료한다.

`Stop-Process -Name rclone`을 쓰지 않는 이유는, 같은 PC에 고객용(18081)과 MSP용(18080) 두 가지 설치본이 있을 때 전체 rclone을 죽이면 다른 설치본도 꺼지기 때문이다. 포트 기준으로 선택적으로 종료한다.

### 6-2. Keystone 토큰 발급

```powershell
$body = @{
    auth = @{
        identity = @{
            methods  = @("password")
            password = @{ user = @{ name = $USERNAME; password = $PASSWORD; domain = @{ id = "default" } } }
        }
        scope = @{ project = @{ id = $PROJECT_ID } }
    }
} | ConvertTo-Json -Depth 10

$response = Invoke-WebRequest -Uri $AUTH_URL -Method POST `
    -ContentType "application/json" -Body $body -UseBasicParsing -ErrorAction Stop
$TOKEN = $response.Headers["X-Subject-Token"]
```

NCP Keystone 서버에 ID/PW를 보내 임시 토큰을 발급받는다.

핵심은 `domain = @{ id = "default" }` 부분이다. `name = "default"`로 보내면 NCP가 인증을 거부한다. NCP Archive Storage는 도메인을 이름이 아닌 **ID 방식**으로만 허용한다.

토큰은 응답 헤더의 `X-Subject-Token`에 담겨온다 (응답 바디가 아니다).

- `ConvertTo-Json -Depth 10` — 중첩된 해시테이블을 JSON으로 변환한다. `-Depth`가 낮으면 깊은 중첩 구조가 잘린다.

### 6-3. rclone.conf 작성

```powershell
$conf = "[ncp-archive]`ntype = swift`nenv_auth = false`nstorage_url = $STORAGE_URL`nauth_token = $TOKEN"
[System.IO.File]::WriteAllText($CONF_PATH, $conf, [System.Text.Encoding]::UTF8)
```

- `` `n `` — PowerShell에서 줄바꿈 문자다.
- `storage_url + auth_token` 조합을 쓰는 이유 — rclone의 일반 Swift 연결 방식은 내부적으로 Keystone 재인증을 시도하는데, NCP의 도메인 ID 방식과 충돌한다. 토큰을 직접 주입하면 rclone이 재인증 없이 해당 토큰을 그대로 사용한다.

### 6-4. rclone HTTP 서버 실행 및 브라우저 오픈

```powershell
$env:RCLONE_CONFIG = $CONF_PATH
Start-Process -FilePath $RCLONE `
    -ArgumentList "serve http ncp-archive: --addr 127.0.0.1:$PORT --no-modtime" `
    -WindowStyle Hidden

Start-Sleep -Seconds 2
Start-Process "http://127.0.0.1:$PORT"
```

- `$env:RCLONE_CONFIG` — rclone이 참조할 설정 파일 경로를 환경변수로 지정한다.
- `serve http ncp-archive:` — rclone을 HTTP 서버 모드로 실행한다. `ncp-archive:`는 rclone.conf에 정의한 스토리지 이름이다.
- `--addr 127.0.0.1:$PORT` — 로컬호스트의 해당 포트에서만 서비스한다. 외부에서는 접근 불가다.
- `--no-modtime` — 파일 수정 시간을 조회하지 않아 속도가 빠르다.
- `-WindowStyle Hidden` — 검은 CMD 창이 뜨지 않도록 숨김 모드로 실행한다.
- `Start-Sleep -Seconds 2` — rclone 서버가 완전히 뜰 때까지 2초 대기한다. 너무 빨리 브라우저를 열면 서버가 아직 준비가 안 돼서 빈 페이지가 나올 수 있다.
- `Start-Process "http://..."` — 기본 브라우저로 해당 URL을 연다.

---

## 7. launcher.vbs 생성

```powershell
$vbsLines = @(
    'Set s = CreateObject("Wscript.Shell")',
    's.Run "powershell.exe -NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -File ""C:\NCP_Archive\launch.ps1""", 0, False'
)
$utf8NoBom = New-Object System.Text.UTF8Encoding $false
[System.IO.File]::WriteAllLines($LAUNCHER_VBS, $vbsLines, $utf8NoBom)
```

VBScript로 PowerShell을 검은 창 없이 실행하는 래퍼다.

- `.Run "...", 0, False` — 두 번째 인자 `0`이 창 숨김 모드다. `1`이면 일반 창, `0`이면 완전히 숨긴다.
- `$utf8NoBom = New-Object System.Text.UTF8Encoding $false` — **BOM 없는 UTF-8**로 파일을 저장한다. `WriteAllText`의 기본 UTF-8은 파일 앞에 BOM(`EF BB BF`) 3바이트를 붙이는데, VBScript는 이 BOM을 파싱하지 못해 `800A0408 유효하지 않은 문자입니다` 오류가 발생한다. `$false`를 넘기면 BOM을 붙이지 않는다.
- `[System.IO.File]::WriteAllLines` vs `WriteAllText` — `WriteAllLines`는 배열의 각 줄을 줄바꿈으로 이어서 쓴다.

---

## 8. 바탕화면 단축키 생성

```powershell
$wsh = New-Object -ComObject WScript.Shell
$lnk = $wsh.CreateShortcut($SHORTCUT)
$lnk.TargetPath       = "C:\Windows\System32\wscript.exe"
$lnk.Arguments        = """C:\NCP_Archive\launcher.vbs"""
$lnk.Description      = "NCP Archive File Browser"
$lnk.WorkingDirectory = $INSTALL_DIR
$lnk.Save()
```

- `-ComObject WScript.Shell` — Windows Shell COM 오브젝트를 통해 바탕화면 단축키(`.lnk` 파일)를 만든다.
- `TargetPath`를 `launcher.vbs`로 직접 지정하지 않는 이유 — `.vbs` 파일을 직접 실행하면 보안 경고가 뜰 수 있다. `wscript.exe`를 실행 대상으로 하고 `.vbs`를 인수로 넘기면 더 안정적이다.
- `"""C:\NCP_Archive\launcher.vbs"""` — 큰따옴표 세 개는 PowerShell 문자열 안에서 큰따옴표 하나를 표현한다. 경로에 공백이 있어도 올바르게 처리되도록 따옴표로 감싼다.

---

## 전체 실행 흐름 요약

```
Install.bat 더블클릭
  └─ Install.ps1 실행
       ├─ [1] C:\NCP_Archive 폴더 생성
       ├─ [2] rclone.exe 설치 (이미 있으면 스킵 / zip 있으면 압축해제 / 없으면 다운로드)
       ├─ [3] launch.ps1 파일 생성 (토큰 발급 + rclone 서버 시작 로직)
       ├─ [4] launcher.vbs 파일 생성 (launch.ps1을 창 없이 실행하는 래퍼)
       └─ [5] 바탕화면에 단축키 생성 → wscript.exe → launcher.vbs → launch.ps1

바탕화면 아이콘 클릭 (매번)
  └─ wscript.exe launcher.vbs (창 없음)
       └─ powershell.exe launch.ps1 (숨김 모드)
            ├─ 기존 rclone 해당 포트 프로세스 종료
            ├─ Keystone 서버에 토큰 발급 요청
            ├─ rclone.conf에 토큰 저장
            ├─ rclone HTTP 서버 백그라운드 실행
            └─ 브라우저 자동 오픈 (http://127.0.0.1:PORT)
```
