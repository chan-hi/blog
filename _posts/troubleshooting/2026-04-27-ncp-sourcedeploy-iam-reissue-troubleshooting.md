---
title: "NCP Source Deploy IAM 키 재발급 후 배포 실패 트러블슈팅"
date: 2026-04-27 15:00:00 +0900
categories: [Tips, Troubleshooting]
tags: [ncp, sourcedeploy, iam, agent, troubleshooting, gov-cloud]
description: "NCP IAM 키 재발급 후 Source Deploy 배포 실패 원인 분석 및 해결 - NCP_AUTH_KEY 변수명 오류, 공공기관용 에이전트 미설치"
---

> NCP IAM 키 재발급 후 Source Deploy 배포가 실패하는 현상의 원인 분석 및 해결 과정을 정리합니다.

## 증상

NCP IAM 키 재발급 후 Source Deploy 배포 실행 시 다음 에러 발생:

```
Failed to connect agent.
Failed to get log from agent.
```

## 환경

- NCP 공공기관용 클라우드 (Gov Cloud)
- 배포 대상: PUSH 서버 1대, WAS 서버 2대 (프라이빗 서브넷)
- Source Deploy Agent `ver=2025.07.17.01`
- OS: CentOS / Python 3.6.8

---

## 원인 1: NCP_AUTH_KEY 변수명 오류

### 에러 로그

```json
{
  "msg": "KeyError: 'NCP_ACCESS_KEY'",
  "type": "error"
}
```

### 원인 분석

sdagent의 `signature.py`는 `/opt/NCP_AUTH_KEY` 파일에서 `NCP_ACCESS_KEY` / `NCP_SECRET_KEY` 키명으로 인증 정보를 읽습니다.

```python
# /opt/sdagent/signature.py
access_key = keys['NCP_ACCESS_KEY']  # 이 키명을 기대
```

그러나 IAM 키 재발급 시 자동 갱신된 `/opt/NCP_AUTH_KEY` 파일의 키명이 다른 형식으로 저장되어 있었습니다.

```bash
# 잘못된 형식 (KeyError 발생)
accesskey=ncp_iam_XXXX
secretkey=ncp_iam_XXXX

# 올바른 형식
NCP_ACCESS_KEY=ncp_iam_XXXX
NCP_SECRET_KEY=ncp_iam_XXXX
```

### 해결 방법

```bash
sudo bash -c 'echo -e "NCP_ACCESS_KEY=발급받은_ACCESS_KEY\nNCP_SECRET_KEY=발급받은_SECRET_KEY" > /opt/NCP_AUTH_KEY'
sudo chmod 400 /opt/NCP_AUTH_KEY
sudo service sourcedeploy restart
```

### 확인

```bash
sudo tail -20 /var/sourcedeploy/log/sourcedeploy-0-system.log
# KeyError 사라지고 아래처럼 나오면 정상
# {"msg": "request retry: 0", "type": "debug"}
```

---

## 원인 2: 민간용 에이전트 설치로 인한 엔드포인트 불일치

### 에러 로그

```json
{
  "msg": "socket.timeout: timed out",
  "type": "error"
}
```

에이전트가 아래 주소로 연결 시도:
```
https://sourcedeploy-agent.apigw.ntruss.com/agent/vpc-v2/poll/deploy
```

### 원인 분석

공공기관용(Gov) NCP 환경에서 **민간용 에이전트**를 설치하면 에이전트가 민간용 Source Deploy 엔드포인트로 통신을 시도합니다. 공공기관용 네트워크에서는 해당 주소에 접근이 불가능하여 `socket.timeout`이 발생합니다.

| 구분 | 에이전트 다운로드 주소 |
|------|----------------------|
| 민간용 (VPC) | `https://sourcedeploy-agent.apigw.ntruss.com/agent/vpc-v2/download/install` |
| 공공기관용 | NCP Gov 콘솔 → Source Deploy → 에이전트 설치 화면에서 확인 |

### 해결 방법

NCP 공공기관용 콘솔에서 제공하는 에이전트 설치 명령어를 사용하여 **공공기관용 에이전트**를 재설치합니다.

```bash
# NCP_AUTH_KEY 올바른 형식으로 작성
sudo bash -c 'echo -e "NCP_ACCESS_KEY=발급받은_ACCESS_KEY\nNCP_SECRET_KEY=발급받은_SECRET_KEY" > /opt/NCP_AUTH_KEY'
sudo chmod 400 /opt/NCP_AUTH_KEY

# 공공기관용 에이전트 설치 (콘솔에서 주소 확인)
wget --header="x-ncp-region_code:KR" "공공기관용_에이전트_다운로드_주소"
chmod 755 install
./install
rm -rf install
```

---

## 트러블슈팅 절차

### 1. 에이전트 로그 위치 확인

```bash
# 시스템 로그 (주요 에러 확인)
sudo tail -100 /var/sourcedeploy/log/sourcedeploy-0-system.log

# 설치 로그
sudo cat /var/sourcedeploy/log/agent.log
```

### 2. 서비스 상태 확인

```bash
sudo service sourcedeploy status
ps aux | grep sdagent
```

### 3. NCP_AUTH_KEY 확인

```bash
sudo cat /opt/NCP_AUTH_KEY
# NCP_ACCESS_KEY= / NCP_SECRET_KEY= 형식인지 확인
```

### 4. 서비스 재시작

```bash
sudo service sourcedeploy restart
sudo tail -20 /var/sourcedeploy/log/sourcedeploy-0-system.log
```

---

## 정상 상태 확인

서비스 재시작 후 로그에 아래처럼 출력되면 정상:

```json
{"msg": "request retry: 0", "type": "debug"}
```

---

## 요약

| 에러 | 원인 | 해결 |
|------|------|------|
| `KeyError: 'NCP_ACCESS_KEY'` | NCP_AUTH_KEY 변수명 오류 | `NCP_ACCESS_KEY=` / `NCP_SECRET_KEY=` 형식으로 수정 |
| `socket.timeout` | 민간용 에이전트를 공공기관용 환경에 설치 | 공공기관용 에이전트로 재설치 |
| `401 Authentication Failed` | IAM 키 권한 부족 | NCP IAM에서 Source Deploy 권한 부여 확인 |
