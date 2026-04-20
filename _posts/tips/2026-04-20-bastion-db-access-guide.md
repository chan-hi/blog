---
title: "경유 서버를 통한 원격 DB 접속 가이드"
date: 2026-04-20 00:00:00 +0900
categories: [Tips, Database]
tags: [ssh, tunnel, mysql, dbeaver, bastion, windows, powershell]
description: "Windows PowerShell 환경에서 경유 서버(Bastion)를 통해 원격 MySQL DB에 SSH 터널로 접속하는 방법"
---

이 문서는 Windows PowerShell 환경에서 경유 서버를 통해 원격 MySQL DB 서버에 접속하는 방법을 안내합니다.

원격 DB 서버가 외부에 직접 열려 있지 않은 경우, 먼저 SSH 터널을 생성한 뒤 DB 접속 도구로 접속해야 합니다.

## 접속 흐름

```text
로컬 PC
  -> SSH 터널 생성
  -> 경유 서버
  -> 원격 MySQL DB 서버
```

실제 DB 접속 도구에서는 원격 DB 주소로 직접 접속하지 않고, 로컬 PC에 생성한 SSH 터널 주소인 `127.0.0.1:<로컬포트>`로 접속합니다.

## 1. 시작 전에 확인할 정보

접속을 시작하기 전에 아래 정보를 전달받아야 합니다.

- 경유 서버 Host, ex) `10.0.0.10`
- 경유 서버 User, ex) `user`
- SSH 키 파일 경로, ex) `C:\keys\example.pem`
- 원격 DB Host, ex) `db.example.internal`
- 원격 DB Port, ex) `3306`
- DB 계정, ex) `dbuser`
- DB 비밀번호

## 2. 사전 준비

아래 항목이 준비되어 있어야 합니다.

- Windows PowerShell 사용 가능
- OpenSSH 클라이언트 설치
- DBeaver 또는 MySQL 클라이언트 사용 가능
- SSH 키 파일 보유

PowerShell에서 SSH 설치 여부 확인:

```powershell
ssh -V
```

## 3. SSH 터널 열기

PowerShell 창을 1개 열고 아래 형식으로 명령을 실행합니다.

```powershell
ssh -N -L <로컬포트>:<원격DB호스트>:<원격DB포트> -i "<키파일경로>" <경유서버계정>@<경유서버호스트>
```

항목 설명:

- `<로컬포트>`: 내 PC에서 사용할 포트, 예시 `13306`
- `<원격DB호스트>`: 접속하려는 원격 DB 서버 주소, 예시 `db.example.internal`
- `<원격DB포트>`: 원격 DB 포트, 예시 `3306`
- `<키파일경로>`: SSH 키 파일 경로, 예시 `C:\keys\example.pem`
- `<경유서버계정>`: 경유 서버 로그인 계정, 예시 `user`
- `<경유서버호스트>`: 경유 서버 주소, 예시 `10.0.0.10`

입력 예시:

```powershell
ssh -N -L 13306:db.example.internal:3306 -i "C:\keys\example.pem" user@10.0.0.10
```

명령 의미:

- `-N`: 원격 셸은 열지 않고 터널만 유지
- `-L`: 로컬 포트를 원격 DB 포트로 전달
- `-i`: SSH 키 파일 지정

정상 동작 시:

- 명령 실행 후 창이 닫히지 않고 그대로 대기합니다
- 이 창은 터널 유지용입니다
- DB 작업이 끝날 때까지 이 창을 닫지 않습니다

## 4. 터널이 열렸는지 확인

새 PowerShell 창을 열고 아래 명령을 실행합니다.

```powershell
netstat -ano | findstr 13306
```

정상일 경우 아래와 비슷한 형태가 표시됩니다.

```text
127.0.0.1:13306
```

또는

```text
0.0.0.0:13306
```

## 5. GUI 도구로 접속하기 (DBeaver)

> SSH 터널이 먼저 열려 있어야 합니다.

주의사항:

- DBeaver의 SSH 탭을 사용하지 말고, 일반 MySQL 연결로 입력합니다
- Host에는 실제 원격 DB 주소가 아니라 `127.0.0.1`을 입력합니다
- Port에는 원격 DB 포트가 아니라 로컬에 열어둔 포트를 입력합니다

### 접속 절차

1. DBeaver 실행
2. `Database` > `New Database Connection` 선택
3. `MySQL` 선택
4. 아래 값 입력 후 `Test Connection` 클릭

| 항목 | 입력값 |
|------|--------|
| Server Host | `127.0.0.1` |
| Port | `13306` (로컬 포트) |
| Database | 비워두거나 전달받은 값 입력 |
| Username | `dbuser` |
| Password | 전달받은 비밀번호 |

5. `Test Connection` 성공 메시지 확인 후 `Finish` 클릭

정상 연결 시:

- `Test Connection` 성공 메시지 표시
- 접속 후 SQL Editor에서 쿼리 실행 가능

## 6. 명령행으로 접속하기

### MySQL CLI

```powershell
mysql -h 127.0.0.1 -P <로컬포트> -u <DB계정> -p
```

입력 예시:

```powershell
mysql -h 127.0.0.1 -P 13306 -u dbuser -p
```

비밀번호 입력 후 정상 접속 시 MySQL 프롬프트가 표시됩니다.

```text
mysql>
```

### MySQL Shell

```powershell
mysqlsh --sql -h 127.0.0.1 -P <로컬포트> -u <DB계정> -p
```

입력 예시:

```powershell
mysqlsh --sql -h 127.0.0.1 -P 13306 -u dbuser -p
```

## 7. 접속 종료

작업이 끝나면 아래 순서로 종료합니다.

1. DBeaver 또는 MySQL 클라이언트를 종료합니다
2. SSH 터널을 열어둔 PowerShell 창으로 이동합니다
3. `Ctrl + C`를 눌러 터널을 종료합니다

## 8. 주의사항

- SSH 키 파일은 외부에 공유하지 않습니다
- DB 비밀번호는 문서에 고정 기재하지 않고 별도 전달합니다
- 터널 창을 닫으면 DB 연결도 함께 끊어집니다
