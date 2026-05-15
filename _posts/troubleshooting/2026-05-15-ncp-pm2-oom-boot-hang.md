---
title: "NCP 서버 부팅 불가 — PM2 OOM 무한 루프 트러블슈팅"
date: 2026-05-15 10:00:00 +0900
categories: [Troubleshooting, NCP]
tags: [NCP, PM2, OOM, Rocky Linux, GRUB, systemd, Next.js, 트러블슈팅]
description: "PM2 reload 중 OOM 발생 → 재부팅마다 kernel hang 반복 → GRUB rescue 모드로 복구한 과정 전체 기록"
---

## 장애 개요

| 항목 | 내용 |
|------|------|
| **발생일** | 2026-05-14 |
| **환경** | Naver Cloud Platform / Rocky Linux 9.6 |
| **영향 범위** | 개발 서버 (서비스 영향 없음) |
| **지속 시간** | 약 2시간 이상 (09:24 ~ 복구 완료) |

개발 서버에 변경사항 merge 후 CI/CD 파이프라인으로 배포를 진행했다. 09:24경 OOM(Out of Memory)으로 배포가 실패하고, 이후 서버가 재부팅해도 정상 부팅이 되지 않는 상태가 지속됐다.

원인은 `pm2-root.service`가 `enabled` 상태로 등록되어 있어, 재부팅마다 메모리가 부족한 상태에서 `mada-next`를 자동으로 다시 기동하는 무한 루프였다.

---

## 타임라인

| 시각 | 이벤트 |
|------|--------|
| **09:11** | 변경사항 merge, CI/CD 배포 시작 |
| **09:24** | 배포 실패 — OOM으로 프로세스 종료 불가 |
| **09:24** | PM2가 구 프로세스(PID 2389242, `mada-next/_old_0`)에 SIGTERM 반복, 5초간 실패 |
| **09:24:41** | SIGKILL 강제 전송, 프로세스 종료 |
| **09:24 이후** | NCP 콘솔에서 Guest Agent "미설치" 표기 시작 |
| **09:24 이후** | 서버 응답 불가 (swap thrashing 상태) |
| **이후** | NCP 콘솔에서 강제 재시작(Hard Restart) 시도 |
| **재시작 후** | 재부팅마다 kernel hang — 정상 부팅 불가 반복 |
| **복구 작업** | GRUB rescue.target 진입, `pm2-root.service` disable 처리 |
| **복구 완료** | 정상 재부팅 후 서버 안정화 확인 |

---

## 원인 분석

### 직접 원인: PM2 reload 시 메모리 부족

PM2의 무중단 배포(`pm2 reload`)는 **신규 인스턴스를 먼저 기동한 후 구 인스턴스를 종료**하는 방식으로 동작한다. 이 과정에서 순간적으로 **기존 + 신규 프로세스가 동시에 메모리를 점유**하여 약 2배의 메모리가 필요하다.

서버 메모리가 이를 감당하지 못해 OOM 상태에 진입했고, 구 프로세스(`mada-next`)가 커널 내 `futex_wait` 상태에서 빠져나오지 못하면서 SIGTERM에 응답하지 않는 D 상태(Uninterruptible Sleep)에 빠졌다.

### 확대 원인: PM2 자동 재시작 무한 루프

`pm2-root.service`가 `enabled` 상태로 등록되어 있어, 부팅 시마다 PM2가 `/root/.pm2/dump.pm2`를 기반으로 `mada-next`를 자동으로 기동했다. 서버 메모리가 `mada-next` 단독 기동조차 감당하기 어려운 상태였으므로 다음 루프가 반복됐다.

```
서버 부팅
  → systemd가 pm2-root.service 기동
  → PM2가 dump.pm2 읽어서 next-server 자동 기동
  → 메모리 부족 → kswapd0 과부하 (swap thrashing)
  → 시스템 응답 불가 수준으로 느려짐
  → kernel이 stuck task 감지 → 커널 스택 트레이스 덤프 반복
  → 부팅 진행 불가 (hung 상태처럼 보임)
```

### Guest Agent 미설치 표기 원인

OOM killer가 메모리 회수 과정에서 NCP Guest Agent 프로세스도 함께 종료한 것으로 추정된다. Agent가 Heartbeat를 NCP 플랫폼에 전송하지 못하면서 콘솔에서 "Guest Agent 미설치"로 표기됐다. 실제로 Agent 바이너리나 설정이 삭제된 것은 아니다.

---

## 로그 및 트레이스 분석

### PM2 로그

```
/root/.pm2/pm2.log last 15 lines:
PM2 | 2026-05-14T09:24:40: PM2 log: pid=2389242 msg=failed to kill - retrying in 100ms
PM2 | 2026-05-14T09:24:40: PM2 log: pid=2389242 msg=failed to kill - retrying in 100ms
...
PM2 | 2026-05-14T09:24:41: PM2 log: Process with pid 2389242 still alive after 5000ms, sending it SIGKILL now...
PM2 | 2026-05-14T09:24:41: PM2 log: pid=2389242 msg=failed to kill - retrying in 100ms
PM2 | 2026-05-14T09:24:41: PM2 log: App name:mada-next id:_old_0 disconnected
PM2 | 2026-05-14T09:24:41: PM2 log: App [mada-next:_old_0] exited with code [0] via signal [SIGKILL]
PM2 | 2026-05-14T09:24:41: PM2 log: pid=2389242 msg=process killed
```

- SIGTERM을 100ms 간격으로 9회 이상 재시도했으나 모두 실패
- 5000ms 경과 후 SIGKILL로 강제 종료
- SIGTERM에 응답 못 한 이유: OOM 상태에서 커널 내 `futex_wait`에 진입해 D 상태(Uninterruptible Sleep)에 빠졌기 때문

### 커널 스택 트레이스

재부팅 후 콘솔에 반복 출력된 hung_task 트레이스:

```
[ 246.566479] <TASK>
[ 246.566777]  __schedule+0x229/0x4a0
[ 246.566878]  ? futex_unqueue+0x38/0x60
[ 246.567911]  schedule+0x2e/0xb0
[ 246.568678]  do_exit+0xae/0x480
[ 246.567498]  do_group_exit+0x2d/0x90
[ 246.567689]  get_signal+0x839/0x860
[ 246.568090]  ? futex_wake+0x401/0x5/0xbfef5
[ 246.568297]  arch_do_signal_or_restart+0x25/0x100
[ 246.568499]  ? do_futex+0x138/0x1d0
...
[ 246.574569]  entry_SYSCALL_64_after_hwframe+0x70/0x80
[ 246.624141] RIP: 0033:0x7f57f360717a
[ 246.625096] RSP: 002b:00007f85b7fdc40  EFLAGS: 00000246  ORIG_RAX: 00000000000000ca
```

- `ORIG_RAX = 0xca` → `__NR_futex` (202번 시스템콜) — futex 대기 중
- `do_group_exit → get_signal → arch_do_signal_or_restart → do_exit`: 시그널 수신 후 종료 경로 진입
- `srso_alias_return_thunk` 반복: AMD CPU의 SRSO 완화 코드(Inception 취약점 패치), 장애 원인이 아님
- RIP가 유저스페이스 — glibc의 futex 래퍼 내부에서 캡처된 트레이스

두 번째 트레이스에서는 `futex_wake` 호출이 여러 번 등장하는데, Node.js 종료 시 libuv 워커 스레드 및 V8 GC 스레드를 정리하는 중임을 의미한다. 동일 패턴이 반복 출력됐다는 것은 PM2 자동 재시작 → OOM → 종료 루프가 재부팅마다 반복되고 있는 것이다.

### 장애 당시 `top` 스냅샷

```
PID   USER  PR  NI    VIRT    RES   SHR S  %CPU  %MEM     TIME+  COMMAND
1783  root  20   0  12.1g  999324     0 S  65.9  26.7   0:07.33  next-server (v1
53    root  20   0   22936   7520  3072 D   5.4   0.2   1:26.02  kswapd0
1782  root  29   9   22936   7520  3072 D   3.9   0.2   0:01.28  systemd-coredump
1720  root  20   0       0      0     0 I   2.5   0.0   0:01.28  kworker/1:0-xfs-conv/vda2
1405  root  20   0 1027560  26700  9984 S   0.5   0.7   0:02.05  PM2 v6.0.14: Go
```

| 프로세스 | 의미 |
|---------|------|
| `next-server` (PID 1783) | CPU 65.9%, MEM 26.7%, VIRT 12.1GB — 부팅 즉시 자동 기동돼 자원 독점 |
| `kswapd0` (PID 53) | CPU 누적 **1분 26초** — 심각한 swap thrashing. 시스템이 극도로 느려진 직접 원인 |
| `systemd-coredump` (PID 1782) | 직전 충돌로 인한 코어 덤프 진행 중 |
| `PM2 v6.0.14` (PID 1405) | dump.pm2 기반으로 next-server 자동 복원 및 재시작 반복 중 |

### systemctl 서비스 상태

rescue 모드 진입 후 확인:

```bash
[root@dev-server ~]# systemctl list-unit-files | grep -i pm2
pm2-root.service         enabled    disabled
```

`pm2-root.service`가 `enabled` 상태 — 부팅 시 자동 기동이 장애의 재발 원인이었다.

---

## 복구 절차

### 시도 1 (실패) — 정상 부팅 상태에서 `pm2 kill`

시스템이 너무 느려 명령 입력 자체가 불가능한 수준이어서 실패했다.

### 시도 2 (실패) — NCP 콘솔 강제 재시작

재부팅마다 동일한 kernel trace가 반복됐다. pm2-root.service가 살아있는 한 재부팅으로는 해결되지 않는 구조였다.

### 시도 3 (성공) — GRUB rescue 모드 진입

**1. NCP 콘솔에서 강제 재시작(Hard Restart)**

**2. VNC 콘솔에서 GRUB 카운트다운 중 ↓ 키로 정지**

**3. 부팅 엔트리 선택 후 `e` 키 입력**

**4. `linux ($root)/vmlinuz-...` 라인 끝에 추가:**

```
systemd.unit=rescue.target
```

**5. `Ctrl+X`로 부팅 → root 비밀번호 입력 → rescue 쉘 진입**

**6. mask 시도 (실패)**

```bash
[root@dev-server ~]# systemctl mask pm2-root
Failed to mask unit: File /etc/systemd/system/pm2-root.service already exists.
```

PM2는 `pm2 startup` 실행 시 unit 파일을 `/etc/systemd/system/`에 직접 설치하기 때문에 mask가 불가능하다. (`mask`는 해당 경로에 `/dev/null` 심볼릭 링크를 만드는 방식인데 파일이 이미 존재해 충돌)

**7. disable로 대체 (성공)**

```bash
[root@dev-server ~]# systemctl disable pm2-root
Removed "/etc/systemd/system/multi-user.target.wants/pm2-root.service".

[root@dev-server ~]# systemctl reboot
```

재부팅 후 `mada-next` 자동 기동 없이 서버 정상 응답 확인 완료.

---

## 근본 원인 요약

| 구분 | 내용 |
|------|------|
| **트리거** | `pm2 reload` 시 신구 인스턴스 동시 점유 → 메모리 2배 필요 |
| **직접 원인** | 가용 메모리 부족 → OOM, 구 프로세스 D 상태 진입 → SIGTERM 무시 |
| **확대 원인** | `pm2-root.service` enabled → 재부팅마다 자동으로 메모리 폭탄 재점화 |
| **복구 지연** | swap 미설정 → OOM 시 완충 없이 시스템 즉시 응답 불가로 악화 |
| **Guest Agent 미표기** | OOM killer에 의해 Guest Agent 종료 → Heartbeat 미전송 |

---

## 재발 방지 대책

### 즉시 조치

**① Swap 공간 추가**

Rocky Linux 9 NCP 이미지는 기본적으로 swap이 미설정된 경우가 많다. swap이 없으면 OOM 발생 시 완충 없이 프로세스가 바로 강제 종료된다.

```bash
fallocate -l 4G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

**② 배포 방식 변경: `pm2 reload` → `pm2 restart`**

`pm2 reload`는 무중단이지만 메모리 2배가 필요하다. 개발 서버에서는 짧은 다운타임을 감수하고 `pm2 restart`를 사용하거나, 파이프라인에서 구 프로세스 종료 후 신규 기동하도록 수정한다.

**③ PM2 ecosystem에 메모리 상한 설정**

```js
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'mada-next',
    script: 'server.js',
    max_memory_restart: '800M',
    instances: 1,
  }]
}
```

### 중기 대책

**④ 서버 메모리 증설**

`next-server` 단독으로 메모리의 26.7%(~1GB)를 점유하고 있어 배포 시 여유가 없다. NCP 스펙 업그레이드 검토.

**⑤ 배포 전 메모리 체크 단계 추가**

```bash
# 가용 메모리가 2GB 미만이면 배포 중단
AVAILABLE=$(awk '/MemAvailable/ {print $2}' /proc/meminfo)
if [ "$AVAILABLE" -lt 2097152 ]; then
  echo "Insufficient memory. Available: ${AVAILABLE}kB. Aborting."
  exit 1
fi
```

**⑥ NCP 서버 스냅샷 정책 수립**

배포 전 자동 스냅샷 트리거를 추가하면 배포 직전 시점으로 즉시 롤백할 수 있다.

**⑦ 메모리 모니터링 알람 설정**

NCP 모니터링 또는 Prometheus + Alertmanager를 통해 메모리 사용률 80% 초과 시 알람 수신.

---

## GRUB rescue 모드 진입 — 상세 가이드

> 이번 장애의 최종 해결 수단. 시스템이 너무 느리거나 부팅 자체가 완료되지 않아 SSH/콘솔 명령이 불가능한 상황에서 사용한다.

### 사전 준비

**VNC 콘솔 미리 열기**

재시작 전에 반드시 VNC 콘솔을 열어두어야 한다. 재시작 후 콘솔을 열려고 하면 GRUB 카운트다운 5초를 놓칠 수 있다.

- NCP 콘솔 → 서버 목록 → 해당 서버 → **"콘솔 연결"** 또는 **"VNC 콘솔"** 클릭
- 새 탭이 열리면 화면 안쪽을 한 번 클릭해서 **키 입력 포커스** 확보

**한글 입력 모드 해제**

VNC 콘솔에서 한글 입력 모드가 켜져 있으면 특수문자(`.`, `=`, `/` 등)가 깨져 입력된다. **한/영 키** 또는 **Shift+Space**로 반드시 영문 모드로 전환.

### GRUB 메뉴 카운트다운 잡기

재시작 직후 GRUB 메뉴가 뜨는 순간 **↓ 키를 한 번** 누른다. 카운트다운이 멈추고 메뉴 선택 상태가 된다.

```
┌────────────────────────────────────────────────────────────┐
│ Rocky Linux (5.14.0-570.18.1.el9_6.x86_64) 9.6 (Blue Onyx) │  ← 선택된 상태
│ Rocky Linux (5.14.0-503.xx.x.el9_6.x86_64) 9.6 (Blue Onyx) │
│ Rocky Linux (0-rescue-xxxxxxxxxxxxxxxxxxxx) 9.6 (Blue Onyx)  │
└────────────────────────────────────────────────────────────┘

  Use the ↑ and ↓ keys to change the selection.
  Press 'e' to edit the selected entry.
```

> GRUB 메뉴가 안 보이고 바로 부팅되는 경우: `GRUB_TIMEOUT_STYLE=hidden` 설정이 돼 있을 수 있다. 부팅 시작 직후 **Esc 키를 빠르게 연타**(0.5초 간격)하거나 **Shift 키를 꾹 누른 채** 부팅 대기.

### 편집 모드 진입 및 파라미터 추가

첫 번째 엔트리(최신 커널)가 선택된 상태에서 **`e` 키**를 누른다.

```
load_video
set gfxpayload=keep
insmod gzio
linux ($root)/vmlinuz-5.14.0-570.18.1.el9_6.x86_64 root=UUID=xxxx-xxxx ro \
crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M biosdevname=0 net.ifnames=0 \
console=ttyS0,115200n8 console=tty0
initrd ($root)/initramfs-5.14.0-570.18.1.el9_6.x86_64.img $tuned_initrd
```

> 줄 끝의 `\`는 GRUB의 시각적 줄바꿈 기호일 뿐, `linux`로 시작하는 내용이 논리적으로 한 줄이다.

`linux` 라인 끝으로 커서를 이동한 후 스페이스 한 칸 + 아래 문자열 입력:

```
systemd.unit=rescue.target
```

입력 후:

```
... console=ttyS0,115200n8 console=tty0 systemd.unit=rescue.target
```

**`Ctrl+X`** 또는 **`F10`**으로 부팅. (`Ctrl+C` 또는 `ESC`는 편집 취소)

> 이 변경은 **일회성**이다. 다음 부팅에는 원래 설정으로 자동 복원된다.

### rescue 쉘 진입

```
You are in rescue mode. After logging in, type "journalctl -xb" to view
system logs, "systemctl reboot" to reboot, "systemctl default" or "exit"
to boot into default mode.

Give root password for maintenance
(or press Control-D to continue):
```

> **절대 `Ctrl+D`를 누르지 않는다.** 일반 부팅을 계속 진행해 PM2가 `mada-next`를 다시 기동하면서 장애 상황이 반복된다.

root 비밀번호 입력 후 쉘 진입. 이 상태에서는 PM2 등 자동 시작 서비스가 구동되지 않아 시스템 자원이 여유롭고 명령 응답이 빠르다.

### rescue 모드에서 수행한 조치

```bash
# 서비스 상태 확인
[root@dev-server ~]# systemctl list-unit-files | grep -i pm2
pm2-root.service         enabled    disabled

# mask 시도 → 실패 (PM2가 unit 파일을 직접 설치해 충돌)
[root@dev-server ~]# systemctl mask pm2-root
Failed to mask unit: File /etc/systemd/system/pm2-root.service already exists.

# disable로 대체 → 성공
[root@dev-server ~]# systemctl disable pm2-root
Removed "/etc/systemd/system/multi-user.target.wants/pm2-root.service".

[root@dev-server ~]# systemctl reboot
```

`disable`은 `multi-user.target.wants/` 디렉토리의 심볼릭 링크(자동 시작 트리거)만 제거하고 unit 파일 자체는 보존한다. PM2 설정, dump.pm2, ecosystem 파일 등 일체 삭제되지 않는다.

나중에 자동 시작을 다시 활성화할 때는:

```bash
systemctl enable pm2-root
```

### mask vs disable 차이

| 구분 | `systemctl disable` | `systemctl mask` |
|------|---------------------|-----------------|
| **동작** | 자동 시작 심볼릭 링크 제거 | unit을 `/dev/null`로 링크 |
| **부팅 시 자동 시작** | 안 됨 | 안 됨 |
| **의존성으로 기동** | 될 수 있음 | 절대 안 됨 |
| **unit 파일 보존** | 보존 | 보존 (원본은 다른 경로) |
| **복원** | `systemctl enable` | `systemctl unmask` |
| **PM2 환경 적합 여부** | ✅ 충분함 | unit 파일 충돌로 사용 불가 |

PM2는 다른 systemd 서비스의 의존성으로 호출되는 경우가 없으므로 `disable`만으로 충분하다.

---

## 유사 장애 대응 체크리스트

- [ ] NCP VNC 콘솔 **재시작 전** 미리 열어두기
- [ ] 콘솔 화면 포커스 클릭 확인
- [ ] 한글 입력 모드 → 영문 전환 확인
- [ ] 강제 재시작 후 GRUB 화면 뜨자마자 **↓ 키**로 카운트다운 정지
- [ ] **`e` 키**로 편집 모드 진입
- [ ] `linux` 라인 끝에 **`systemd.unit=rescue.target`** 추가
- [ ] **`Ctrl+X`**로 부팅 (`F10`도 동일)
- [ ] root 비밀번호 입력 (`Ctrl+D` 절대 금지)
- [ ] 쉘 진입 후 원인 서비스 `systemctl disable` 처리
- [ ] `systemctl reboot`으로 정상 재부팅
- [ ] 재부팅 후 `free -h`, `systemctl status pm2-root` 로 정상화 확인
