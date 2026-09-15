# RHCSA Week 03 퀴즈

> 총 10문항 · 파트별 5문항 · 권장 5~10분 · 문항당 정답 1개

---

## PART 1. 네트워크·Hostname / DNF·RPM

> RHEL 10 기준. 필요한 도구·권한과 문제에 명시한 계정·파일·프로필은 준비되어 있다고 가정합니다.

### 1. 기존 연결 프로필 `lab`에 IPv4와 IPv6 주소를 모두 수동으로 지정하려고 합니다. 두 빈칸에 들어갈 값의 조합은 무엇입니까?

```bash
sudo nmcli con mod lab ipv4.addresses 192.0.2.10/24 \
  ipv4.method ____ ipv6.addresses 2001:db8::10/64 \
  ipv6.method ____
```

- A. `auto / auto`
- B. `manual / manual`
- C. `disabled / manual`
- D. `manual / disabled`

---

### 2. `nmcli`로 시스템의 지속적인 호스트명을 `servera.lab.example`로 설정하는 명령은 무엇입니까?

- A. `sudo nmcli con up servera.lab.example`
- B. `sudo nmcli device hostname servera.lab.example`
- C. `sudo nmcli general status servera.lab.example`
- D. `sudo nmcli general hostname servera.lab.example`

---

### 3. RHEL 10에서 DNF가 사용할 사용자 지정 RPM 리포지토리의 `.repo` 파일을 일반적으로 저장하는 디렉터리는 어디입니까?

- A. `/etc/yum.repos.d/`
- B. `/etc/ssh/`
- C. `/var/log/`
- D. `/etc/sudoers.d/`

---

### 4. 활성 리포지토리에서 `httpd`와 필요한 의존 패키지를 함께 설치하려고 합니다. 가장 적절한 명령은 무엇입니까?

- A. `sudo rpm -e httpd`
- B. `rpm -q httpd`
- C. `sudo dnf install httpd`
- D. `sudo dnf remove httpd`

---

### 5. 이미 설치된 `/usr/bin/ssh` 파일을 제공한 RPM 패키지를 조회하는 명령은 무엇입니까?

- A. `rpm -qa /usr/bin/ssh`
- B. `rpm -qf /usr/bin/ssh`
- C. `rpm -ql /usr/bin/ssh`
- D. `rpm -qi /usr/bin/ssh`

---

## PART 2. Flatpak / SSH 서비스·키 기반 인증

### 6. Flatpak 원격 리포지토리를 추가하는 아래 명령의 의미로 올바른 것은 무엇입니까?

URL은 유효한 `.flatpakrepo` 파일 주소라고 가정합니다.

```bash
flatpak remote-add --user --if-not-exists labrepo URL
```

- A. 모든 사용자를 위한 원격 리포지토리를 강제로 교체한다.
- B. 현재 사용자의 `labrepo` 애플리케이션을 삭제한다.
- C. 현재 사용자가 설치한 모든 앱을 업데이트한다.
- D. 현재 사용자 범위에 추가하며, 같은 이름이 있으면 추가를 건너뛴다.

---

### 7. 현재 사용자 범위의 `labrepo`에서 `APP_ID` 앱을 설치하고, 나중에 그 앱을 제거하는 명령 조합은 무엇입니까?

`APP_ID`는 해당 리포지토리의 유효한 앱 ID입니다.

- A. 설치: `flatpak install --user labrepo APP_ID` / 제거: `flatpak uninstall --user APP_ID`
- B. 설치: `flatpak remote-add --user labrepo APP_ID` / 제거: `flatpak remote-ls --user labrepo`
- C. 설치: `flatpak list --user` / 제거: `flatpak update --user APP_ID`
- D. 설치: `dnf install APP_ID` / 제거: `rpm -e APP_ID`

---

### 8. `openssh-server`가 설치된 RHEL 10에서 SSH 서비스를 지금 시작하고 부팅 시 자동 시작하도록 함께 설정하는 명령은 무엇입니까?

- A. `sudo systemctl start sshd`
- B. `sudo systemctl enable sshd`
- C. `sudo systemctl enable --now sshd`
- D. `sudo systemctl disable --now sshd`

---

### 9. 일반적인 SSH 공개 키 인증에서 개인 키와 공개 키의 배치로 올바른 것은 무엇입니까?

- A. 개인 키는 접속하는 클라이언트에 보관하고, 공개 키는 서버의 대상 계정 `authorized_keys`에 등록한다.
- B. 개인 키를 서버 `authorized_keys`에 등록하고 공개 키는 삭제한다.
- C. 공개 키와 개인 키를 모두 서버의 `/etc/hosts`에 등록한다.
- D. 서버에 대상 사용자 계정이 없어도 공개 키만 있으면 로그인할 수 있다.

---

### 10. 서버 대상 계정의 `~/.ssh`와 `authorized_keys` 권한을 점검하려고 합니다. 일반적으로 권장되는 권한 조합은 무엇입니까?

두 경로의 소유자는 대상 사용자입니다.

- A. `~/.ssh`: `777` / `authorized_keys`: `666`
- B. `~/.ssh`: `700` / `authorized_keys`: `600`
- C. `~/.ssh`: `600` / `authorized_keys`: `700`
- D. `~/.ssh`: `755` / `authorized_keys`: `777`

---

