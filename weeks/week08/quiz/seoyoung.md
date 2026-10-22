# Quiz

## 개념 확인 (GRUB / 부팅 타겟 / rd.break)

---

**1.** `/boot/grub2/grub.cfg`를 직접 수정해서 커널 인자를 추가했고, 부팅도 잘 됐다. 이후 `grub2-mkconfig -o /boot/grub2/grub.cfg`를 다시 실행하면 어떻게 되는가?

① 직접 수정한 내용이 그대로 유지된다

② 수정한 내용이 `/etc/default/grub`에 자동으로 반영된다

③ `/etc/default/grub` 기준으로 다시 생성되어 수정한 내용이 사라진다

④ 수정한 내용이 `grubenv`로 옮겨진다

---

**2.** 재부팅 없이 **지금 바로** rescue 상태로 전환하고 싶다. 단, 다음 부팅의 기본 타겟은 **그대로** 두어야 한다. 알맞은 명령은?

① `systemctl isolate rescue.target`

② `systemctl set-default rescue.target`

③ `grubby --update-kernel=ALL --args="systemd.unit=rescue.target"`

④ `systemctl get-default`

---

**3.** root 비번을 모르는 상태에서 `rd.break`는 root 셸을 열 수 있지만, `rescue.target`은 root 비번을 요구한다. 그 이유로 알맞은 것은?

① `rd.break`는 진짜 루트가 읽기 전용으로 마운트되기 때문이다

② `rd.break`는 SELinux가 꺼진 상태에서 동작하기 때문이다

③ `rd.break`는 커널 인자로 지정하기 때문이다

④ `rd.break`는 진짜 루트로 갈아타기 전, 인증이 붙기 전 단계에서 멈추기 때문이다

---

**4.** `rd.break`로 멈춘 셸에서 root 비번을 바꾸려 한다. 올바른 순서는?

① `passwd root` → `mount -o remount,rw /sysroot` → `chroot /sysroot` → `touch /.autorelabel`

② `mount -o remount,rw /sysroot` → `chroot /sysroot` → `passwd root` → `touch /.autorelabel`

③ `chroot /sysroot` → `passwd root` → `mount -o remount,rw /sysroot` → `touch /.autorelabel`

④ `touch /.autorelabel` → `passwd root` → `chroot /sysroot` → `mount -o remount,rw /sysroot`

---

## 정답 및 해설

**1. ③ `/etc/default/grub` 기준으로 다시 생성되어 수정한 내용이 사라진다**

- `grub.cfg`는 사람이 고치는 파일이 아니라 **원본에서 생성된 결과물**이다.
- `grub2-mkconfig`는 `/etc/default/grub`을 읽어 `grub.cfg`를 새로 만들므로, 직접 수정한 내용은 덮어써진다.

```
/etc/default/grub (원본) ──grub2-mkconfig──▶ /boot/grub2/grub.cfg (결과물)
```

| 하고 싶은 것 | 방법 |
| --- | --- |
| 영구 설정 변경 | `/etc/default/grub` 수정 → `grub2-mkconfig` |
| 커널 인자 영구 변경 | `grubby --update-kernel=ALL --args="옵션"` |
| 이번 부팅만 변경 | GRUB 메뉴에서 `e` → `Ctrl+X` |

---

**2. ① `systemctl isolate rescue.target`**

- `isolate`는 **지금 상태만** 전환하고, 기본 타겟은 그대로 둔다.
- `set-default`는 **다음 부팅부터** 적용되는 기본값을 바꾼다. ②는 앞으로 부팅할 때마다 rescue로 올라오게 된다.
- ③도 커널 인자를 영구로 저장하므로 매 부팅이 rescue로 간다.
- ④는 현재 기본값을 **확인**만 하는 명령이다.

| 명령 | 바뀌는 것 |
| --- | --- |
| `isolate` | 지금 시스템 상태 |
| `set-default` | 다음 부팅부터의 기본값 |

---

**3. ④ 진짜 루트로 갈아타기 전, 인증이 붙기 전 단계에서 멈추기 때문이다**

```
펌웨어 → GRUB → 커널 → initramfs → systemd → 타겟
                         ▲             ▲
                     rd.break     rescue / emergency
                   (인증 없음)      (root 비번 요구)
```

| 방법 | 단계 | root 비번 요구 |
| --- | --- | --- |
| `rd.break` | initramfs | X |
| `rescue.target` | systemd | O |
| `emergency.target` | systemd | O |
- ①은 사실이지만 원인이 아니다. 읽기 전용인 것과 인증 여부는 별개다.
- ②와 ③도 인증 여부와 무관하다.
- 비번을 모를 때 `rd.break`가 필요한 이유는 **인증 이전 단계에서 멈추기 때문**이다.

---

**4. ② `mount -o remount,rw /sysroot` → `chroot /sysroot` → `passwd root` → `touch /.autorelabel`**

| 순서 | 명령 | 이유 |
| --- | --- | --- |
| 1 | `remount,rw` | `/sysroot`가 읽기 전용이라 `/etc/shadow`를 수정할 수 없다 |
| 2 | `chroot /sysroot` | 지금 `/`는 임시 루트다. 진짜 루트를 `/`처럼 만들어야 진짜 `/etc/shadow`가 수정된다 |
| 3 | `passwd root` | `/etc/shadow`의 root 해시를 새 비번으로 교체한다 |
| 4 | `touch /.autorelabel` | initramfs에서는 SELinux 정책이 로드되지 않아 라벨이 틀어질 수 있다. 다음 부팅 때 정책이 로드된 뒤 전체 relabel을 하도록 예약한다 |
- ①은 읽기 전용 상태에서 `passwd`를 먼저 해서 실패하고, `chroot` 전이라 임시 루트의 `/etc`를 건드리게 된다.
- ③은 `chroot`를 한 뒤에 `/sysroot`를 remount하려 해서 경로가 맞지 않는다. remount는 `chroot` 전에 해야 한다.
- ④는 `.autorelabel`을 임시 루트에 만들게 되어 진짜 루트에는 예약되지 않는다.
- 마지막에 `exit`을 두 번 한다. 첫 번째는 chroot에서 나오고, 두 번째는 initramfs 셸을 끝내서 부팅을 이어간다.