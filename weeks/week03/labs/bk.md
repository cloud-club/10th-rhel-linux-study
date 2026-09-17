# Week 03 - Lab

## Flatpak 리포지토리 구성 및 SSH 키 기반 인증

`servera` 시스템에 `admin` 사용자로 로그인합니다. (`admin@servera`)

`servera` 시스템에서 `serverb` 시스템에 `admin` 사용자로 SSH 로그인합니다. (`admin@serverb`)

두 번째 터미널을 열어 `servera` 시스템에서 `admin` 사용자에 대한 SSH 키 페어를 생성합니다.

> 암호(passphrase)는 설정하지 마십시오.

`admin` 사용자의 SSH 공개키를 `serverb` 시스템의 `admin` 사용자에게 전송합니다.

`serverb` 시스템에서 공개키가 정상적으로 등록되었는지 확인합니다.

첫 번째 터미널로 전환해 `serverb` 시스템에서 암호로 인증할 수 없도록 `sshd` 서비스를 구성합니다.

두 번째 터미널로 전환해 `servera` 시스템에서 SSH 키를 사용하여 `serverb` 시스템에 성공적으로 로그인할 수 있는지 확인합니다. (`admin@serverb`)

첫 번째 터미널로 전환해 `serverb` 시스템에 `root` 사용자로 로그인하지 못하도록 `sshd` 서비스를 구성합니다.

두 번째 터미널로 전환해 `servera` 시스템에서 `serverb` 시스템의 `root` 사용자로 로그인할 수 없는지 확인합니다. (`root@serverb`)