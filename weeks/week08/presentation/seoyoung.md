# 개념 정리

- GRUB 부트로더 수정 및 기본 부팅 타겟(Target) 변경
- 부팅 프로세스 중단 및 rd.break 진입을 통한 Root 패스워드 재설정(check v10부터 아마 rd.break이 안된다는…!?)

# 부팅 프로세스 & root 비번 복구 개념 정리

---

## 1. 부팅 프로세스

```
펌웨어(UEFI) → GRUB → 커널 → initramfs → systemd → 타겟
```

- **펌웨어**
    - 메인보드에 내장된 소프트웨어
    - 하드웨어를 점검하고 디스크에서 부트로더(GRUB)를 찾는다.
- **GRUB**
    - 부트로더
    - 커널과 initramfs를 디스크에서 메모리에 올린다.
    - 커널에 넘길 **인자**(루트 디스크 위치, 옵션)도 같이 전달
    
    펌웨어는 똑똑하지 않아서 리눅스 파일시스템을 읽을 줄 모름 → 커널을 직접 올릴 수 없다
    
    그래서 펌웨어가 "디스크에서 이 작은 프로그램만 실행해줘"라고 하면, 그 프로그램(부트로더)이 파일시스템을 읽어서 커널을 메모리에 올림
    
- **커널**
    - GRUB이 메모리에 올려준 뒤부터 실행됨
    - 하드웨어를 인식하고, 메모리와 프로세스를 관리하는 OS의 핵심 기능을 시작
    - 이 시점엔 아직 진짜 루트 파일시스템이 없어서, 메모리에 올라온 initramfs를 임시 루트로 쓰고 거기서 다음 단계를 실행
- **initramfs**
    - 임시 루트 파일시스템
    - 진짜 루트를 마운트하는 데 필요한 도구가 들어 있고, 마운트가 끝나면 진짜 루트로 갈아탄다.
        - 루트 파일시스템의 `/` 아래에 드라이버 파일이 있고, 그걸 읽어야 루트 파일시스템을 마운트할 수 있는데, 읽으려면 이미 마운트가 돼 있어야 해서 순환이 생긴다.
        - 그래서 **루트를 마운트하는 데 필요한 최소한의 것만** 골라서 initramfs에 모아두고, 그걸 메모리에 올려서 먼저 사용.
- **systemd**
    - PID 1, 첫 번째 프로세스
    - 루트 파일시스템 **안에 설치돼 있는** 프로그램으로,
    - 루트가 마운트된 다음(갈아탄 다음)에야 실행됨
    - 지정된 타겟을 보고, 그 타겟에 속한 서비스들을 의존 관계 순서대로 기동
- target
    - systemd가 "여기까지 올려라" 하고 정해주는 **목표 상태**

---

## 2. GRUB (부트로더)

- 부트로더 = 커널을 메모리에 올려주는 프로그램, 커널 인자도 함께 전달
- 펌웨어는 리눅스 파일시스템을 읽지 못하므로, 부트로더가 대신 커널을 올린다.

#### 영구 설정

1. `/etc/default/grub`을 수정한다.
2. `grub2-mkconfig -o /boot/grub2/grub.cfg`로 반영한다.
3. `grub.cfg`에서 원본(`/etc/default/grub`)으로 생성된 결과물을 확인할 수 있다.

| 파일 | 정체 | 지속성 |
| --- | --- | --- |
| `/etc/default/grub` | 사람이 수정하는 **원본** 설정 | 영구 |
| `grub.cfg` | 원본으로부터 **생성된 결과물**, GRUB이 부팅 때 실제로 읽는 파일 | 영구 (직접 수정 X, 재생성하면 덮어써짐) |

**`/etc/default/grub`에서 자주 만지는 항목**

| 항목 | 의미 |
| --- | --- |
| `GRUB_TIMEOUT` | 부팅 메뉴 대기 시간(초) |
| `GRUB_DEFAULT` | 기본으로 부팅할 항목(여러 커널 중 어떤 걸로 기본 부팅할지 정하는 설정) |
| `GRUB_CMDLINE_LINUX` | 커널 인자 |
- `GRUB_DEFAULT`는 숫자(`0`)나 항목 이름을 직접 적을 수도 있고, `saved`를 적을 수도 있다.
    - `saved`는 "저장된 값을 따라가라"는 뜻이다. 실제 선택은 `/boot/grub2/grubenv`에 저장되어 있다.
    - `saved`일 때는 `/etc/default/grub`을 고치고 `grub2-mkconfig`를 해도 기본 커널이 바뀌지 않는다. 아래 명령을 쓴다.
    
    ```bash
    grubby --default-kernel            # 현재 기본 커널 확인
    grubby --set-default <커널경로>    # 기본 커널 변경
    grub2-editenv list                 # grubenv에 저장된 값 확인
    ```
    

#### `grubby`

- 부팅 항목(커널 인자, 기본 커널)을 명령어로 수정하는 도구
- `grub2-mkconfig` 없이 바로 반영되고 **영구**
- RHEL 8 이후는 커널마다 부팅 항목이 개별 파일(`/boot/loader/entries/*.conf`)로 나뉘어 있고, `grubby`가 이 파일을 직접 수정한다.

```bash
grubby --info=ALL                                 # 부팅 항목과 인자 확인
grubby --default-kernel                           # 기본 부팅 커널
grubby --update-kernel=ALL --args="옵션"          # 인자 추가
grubby --update-kernel=ALL --remove-args="옵션"   # 인자 제거
```

- `-update-kernel=ALL` = 설치된 **모든 커널**(같은 OS의 커널 버전 여러 개)에 적용한다.
- 커널은 업데이트하면 추가로 설치되고, 기본으로 최근 3개까지 유지된다 (`installonly_limit`).
- **`-args`에 넣는 커널 인자**

| 인자 | 의미 |
| --- | --- |
| `rd.break` | initramfs 단계에서 멈춤 |
| `systemd.unit=rescue.target` | 이번 부팅의 타겟 지정 |
| `init=/bin/bash` | systemd 대신 bash를 첫 프로세스로 실행 |
| `rhgb quiet` | 그래픽 부팅과 메시지 숨김 (지우면 부팅 로그가 보인다) |
| `selinux=0` / `enforcing=0` | SELinux 끔 / permissive 모드 |
| `console=ttyS0` | 시리얼 콘솔로 출력 |
| `net.ifnames=0` | 네트워크 이름을 `eth0` 방식으로 |
- `rd.break`, `init=/bin/bash`는 보통 `grubby`로 영구 설정하지 않고 `e`로 일회성으로 넣는다.

#### 임시 설정

- 부팅 시 GRUB 메뉴에서 `e` → `Ctrl+X`
- 이번 부팅 한 번만 적용된다. 수정 내용은 메모리에서만 쓰이고 디스크에는 저장되지 않는다.

#### 정리

| 방식 | 지속성 |
| --- | --- |
| `/etc/default/grub` + `grub2-mkconfig` | 영구 |
| `grubby` | 영구 |
| 부팅 시 GRUB 메뉴에서 `e` → `Ctrl+X` | 이번 부팅 한 번만 |

---

## 3. 기본 부팅 타겟

- 타겟 = 부팅이 끝났을 때 도달할 상태 (서비스 묶음)

| 타겟 | 의미 | 예전 런레벨 |
| --- | --- | --- |
| `graphical.target` | GUI 포함 | 5 |
| `multi-user.target` | CLI, 멀티유저 | 3 |
| `rescue.target` | 단일 사용자 복구 | 1 |
| `emergency.target` | 최소 환경 | - |

```bash
systemctl get-default                    # 현재 기본값
systemctl set-default multi-user.target  # 다음 부팅부터 적용 (영구)
systemctl isolate rescue.target          # 지금 즉시 전환 (기본값 그대로)
```

- `set-default` = 기본값을 바꾼다, `isolate` = 지금 상태만 바꾼다.

---

## 4. 부팅 중단 (rd.break)

`rd.break` = initramfs 단계, 진짜 루트로 넘어가기 직전에 멈춘다.

#### 흐름

1. initramfs가 드라이버로 진짜 디스크를 읽어서 `/sysroot`에 **마운트**
2.  `/`를 임시 루트에서 `/sysroot`로 **갈아탄다(switch_root)**.
3. systemd 실행.

`rd.break`는 **1번과 2번 사이 즉,** `rd.break`는 진짜 루트가 `/sysroot`에 마운트된 직후, `/`로 갈아타기 직전에 멈춘다 → 마운트는 끝났지만 아직 `/`로  갈아타지는 않은 상태

이 시점엔 인증이 없어서 root 셸이 바로 열린다.

진짜 루트는 `/sysroot`에 **읽기 전용**으로 마운트되어 있다.

| 방법 | 단계 | root 비번 요구 |
| --- | --- | --- |
| `rd.break` | initramfs | X |
| `rescue.target` | systemd | O |
| `emergency.target` | systemd | O |
- 비번을 모를 때는 인증 전에 멈추는 `rd.break`만 쓸 수 있다.

**진입 방법**

1. 재부팅 후 GRUB 메뉴에서 `e`
2. `linux` 줄 끝에 `rd.break` 추가
3. `Ctrl+X`로 부팅

---

## 5. root 비번 재설정

```bash
mount -o remount,rw /sysroot   # 읽기 전용으로 마운트된 걸 쓰기 가능으로 다시 마운트
chroot /sysroot                # /sysroot를 /처럼 사용
passwd root                    # root 비번 변경
touch /.autorelabel            # 다음 부팅 때 파일을 발견하고 모든 파일의 라벨을 정책대로 리라벨
exit                           # chroot 나가기
exit                           # initramfs 셸 종료 → 부팅 이어서 진행
```

| 명령 | 이유 |
| --- | --- |
| `remount,rw` | `/sysroot`가 읽기 전용이라 `/etc/shadow`를 못 바꾼다 |
| `chroot` | 지금 `/`는 임시 루트다. 진짜 루트를 `/`로 만들어야 `passwd`가 진짜 `/etc/shadow`를 수정한다 |
| `autorelabel` | initramfs에서는 SELinux 정책이 로드되지 않아 `/etc/shadow`의 컨텍스트가 틀어질 수 있다. 틀어지면 로그인이 거부된다 |

#### **흐름**

1. `rd.break` 단계: 정책이 로드 전이라 라벨을 못 붙임. 그래서 `passwd` 후 `.autorelabel` 파일만 만들어둠.
2. `exit` 두 번 → 갈아타기 → systemd가 부팅을 이어가면서 **SELinux 정책이 로드됨.**
3. 부팅 초반, 정책이 로드된 뒤에 시스템이 `/.autorelabel`을 발견하고 **그때 전체 relabel을 실행**. 끝나면 이 파일을 지우고 재부팅.
4. 재부팅 후에는 라벨이 올바른 상태로 정상 로그인됨

---

### 요약

- 부팅은 `펌웨어 → GRUB → 커널 → initramfs → systemd → 타겟` 순으로 진행된다.
- GRUB에서 커널 인자를, systemd에서 기본 타겟을 바꿀 수 있다.
- 루트 비번을 모를 땐 로그인 인증이 붙기 전인 initramfs 단계에서 `rd.break`로 멈춰 `/sysroot`를 rw로 remount하고 `chroot`한 뒤 `passwd`로 비번을 바꾼다.
- 마지막에 `touch /.autorelabel`로 다음 부팅 때 SELinux relabel을 예약한다.