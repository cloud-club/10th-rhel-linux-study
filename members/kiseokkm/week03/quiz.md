# Week 03 - Quiz

## 답안

| 문제 | 답 | 간단 코멘트 |
| --- | --- | --- |
| 1 | B | IPv4와 IPv6를 수동으로 설정하려면 `ipv4.method`, `ipv6.method`를 모두 `manual`로 지정해야 합니다. |
| 2 | D | `nmcli general hostname` 뒤에 새 호스트명을 지정하면 시스템의 호스트명을 설정할 수 있습니다. |
| 3 | A | RPM 리포지토리의 `.repo` 파일은 일반적으로 `/etc/yum.repos.d/`에 저장됩니다. |
| 4 | C | `dnf install`은 활성 리포지토리에서 패키지와 필요한 의존성을 함께 해결하여 설치합니다. |
| 5 | B | `rpm -qf`는 지정한 파일이 어느 설치된 RPM 패키지에 속하는지 확인할 때 사용합니다. |
| 6 | D | `remote-add`는 리포지토리를 추가하고, `--user`는 현재 사용자 범위, `--if-not-exists`는 같은 이름이 있으면 추가를 건너뜁니다. |
| 7 | A | Flatpak 앱은 `install`로 설치하고 `uninstall`로 제거하며, 사용자 범위에서는 `--user`를 함께 사용합니다. |
| 8 | C | `systemctl enable --now sshd`는 서비스를 즉시 시작하면서 부팅 시 자동 시작도 함께 설정합니다. |
| 9 | A | 개인 키는 클라이언트에 보관하고, 공개 키는 서버 대상 계정의 `~/.ssh/authorized_keys`에 등록합니다. |
| 10 | B | 일반적으로 `~/.ssh`는 `700`, `authorized_keys`는 `600` 권한을 사용합니다. |

---