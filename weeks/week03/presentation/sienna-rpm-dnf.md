# RPM을 사용하여 소프트웨어 설치 및 업데이트

RH124 12장 (교재 328–374p)

---

## 1. RPM

### 왜 패키지인가

소프트웨어를 tar로 풀어서 쓰면 이런 게 안 됩니다.

- 이 파일이 어디서 왔는지 모른다
- 지울 때 뭘 지워야 하는지 모른다
- 이 프로그램이 뭘 필요로 하는지 모른다

RPM은 이걸 전부 기록해둡니다. 설치된 패키지 정보는 로컬 RPM 데이터베이스에 저장됩니다.

Red Hat은 RHEL용 소프트웨어를 전부 RPM으로 제공합니다.

### 파일 이름 4요소

```
coreutils-9.5-6.el10.aarch64.rpm
--------- --- ---- -------
  name    ver  rel   arch
```

| 요소 | 의미 |
|---|---|
| name | 패키지 이름 (`coreutils`) |
| version | 원본 소프트웨어의 버전 (`9.5`) |
| release | 같은 버전을 몇 번째로 패키징했는가 (`6.el10`) |
| arch | 컴파일된 아키텍처 (`aarch64`, `x86_64`, `noarch`) |

version과 release는 정하는 주체가 다릅니다.
version은 원저자가 정하고, release는 패키지를 만든 사람이 정합니다. 소스는 그대로인데 빌드 옵션이나 패치만 바꿔서 다시 말면 release가 올라갑니다.

`el10` = Enterprise Linux 10. `noarch`는 아키텍처를 안 타는 것(스크립트, 문서, 폰트 등).

리포지토리에서 설치할 때는 이름만 주면 됩니다. 여러 개 있으면 버전 높은 것, 버전이 같으면 release 높은 것이 선택됩니다.

### RPM 안에 든 것

```
┌──────────────────┐
│ 헤더(메타데이터)  │  이름, 버전, 의존성, 라이선스, 변경로그
├──────────────────┤
│ 페이로드          │  압축된 cpio 아카이브 = 실제 설치될 파일들
├──────────────────┤
│ 스크립트          │  설치 전/후, 삭제 전/후에 실행됨
└──────────────────┘
```

스크립트가 포함된다는 것은 설치할 때 root 권한으로 임의 코드가 실행된다는 뜻입니다. 출처가 불분명한 RPM을 설치하면 안 되는 이유입니다.

---

## 2. rpm 명령으로 조사하기

`rpm`은 **이미 설치된 것**의 로컬 DB를 보는 도구입니다. 질의는 전부 `-q`로 시작합니다.

```bash
rpm -qa                     # 설치된 패키지 전부
rpm -q nmap                 # 이 패키지 설치돼 있나? 버전은?
rpm -qi nmap                # 상세 정보
rpm -ql nmap                # 이 패키지가 설치한 파일 전부
rpm -qc openssh-server      # 설정 파일만
rpm -qd openssh-server      # 문서(man 등)만
rpm -qf /etc/ssh/sshd_config    # 이 파일은 누가 설치했나
```

### `-qf` — 파일에서 패키지 역추적

```bash
$ rpm -qf /etc/yum.repos.d
rocky-release-10.0-...
```

설치된 파일이 어느 패키지에서 왔는지 확인할 때 씁니다.

### `-p` — 설치 전 패키지 파일 조회

```bash
rpm -qlp nmap-7.92-5.el10.aarch64.rpm
```

`-p`가 붙으면 로컬 DB가 아니라 그 파일을 직접 엽니다. 설치 전에 내용을 확인할 때 씁니다.

### 스크립트 보기

```bash
rpm -q --scripts openssh-server
```

sshd 계정을 UID 74로 만드는 `useradd`가 들어있습니다.

```bash
getent group 'sshd' || groupadd -f -g '74' -r 'sshd'
useradd -r -u '74' -g 'sshd' -s '/usr/sbin/nologin' ... 'sshd'
```

2주차에서 본 시스템 계정 UID 1~200이 여기서 만들어집니다.

### 설치와 추출

```bash
rpm -ivh nmap-7.92-5.el10.aarch64.rpm
#  -i 설치  -v 상세출력  -h 진행표시(#####)
```

단, 의존성을 자동으로 해결하지 못합니다. 필요한 패키지가 없으면 실패합니다.

설치하지 않고 파일만 꺼내려면:

```bash
rpm2cpio nmap-7.92-5.el10.aarch64.rpm | cpio -idv
```

RPM 안에 든 cpio 아카이브를 꺼내주는 겁니다. 설정 파일 하나만 복구할 때 정도 씁니다.

---

## 3. rpm과 dnf

```
rpm    패키지 파일 하나를 다루는 저수준 도구
       로컬 DB 조회, 단일 파일 설치
       리포지토리를 모르고 의존성을 못 푼다

dnf    리포지토리에서 받아오고 의존성까지 해결하는 상위 도구
       내부적으로는 rpm을 쓴다
```

RHEL 9부터 yum 대신 dnf가 기본입니다. yum은 호환용 심볼릭 링크로만 남아있습니다.

```bash
$ ls -l /bin/yum
yum -> dnf-3
```

rpm은 패키지 하나를 다루고, dnf는 리포지토리에서 가져옵니다.

---

## 4. BaseOS와 AppStream

RHEL 8부터 리포지토리가 둘로 갈렸습니다.

```
BaseOS      OS 핵심. 커널, systemd, glibc, 셸
            RHEL 수명(약 10년) 내내 버전 고정

AppStream   그 위의 응용. httpd, python, postgresql, nmap
            각자 수명주기가 따로. 더 짧을 수 있음
```

분리한 이유는 다음과 같습니다.

예전에는 한 덩어리라서 Python 2.6 같은 게 OS에 박히면 10년간 못 올렸습니다. 그런데 요구가 정반대인 두 집단이 있습니다.

- 운영자: 커널과 glibc는 절대 안 바뀌었으면 좋겠다
- 개발자: 런타임은 최신을 쓰고 싶다

한 덩어리로는 둘 다 만족이 안 돼서 분리했습니다.

Rocky Linux에는 `extras`가 하나 더 있습니다.

```bash
$ dnf repolist
repo id       repo name
appstream     Rocky Linux 10 - AppStream
baseos        Rocky Linux 10 - BaseOS
extras        Rocky Linux 10 - Extras
```

---

## 5. dnf로 찾고 설치하기

### 찾기

```bash
dnf list http*                # 이름으로 (와일드카드)
dnf search KEYWORD            # 이름 + 요약에서
dnf search all 'web server'   # 이름 + 요약 + 설명까지
dnf info nmap                 # 상세 정보
dnf provides /var/www/html    # 이 파일을 제공하는 패키지는?
```

### `rpm -qf` vs `dnf provides`

둘 다 "이 파일이 어느 패키지 것이냐"인데 보는 곳이 다릅니다.

```
rpm -qf        이미 설치된 파일만.        로컬 RPM DB를 봄
dnf provides   아직 설치 안 한 것도 찾음.  리포지토리 메타데이터를 봄
```

"`nmap` 명령을 쓰고 싶은데 없다" → `dnf provides`
"이 설정 파일이 어디서 왔지" → `rpm -qf`

### 설치 / 업데이트 / 제거

```bash
dnf install nmap
dnf update nmap       # 이름 생략하면 전체 업데이트
dnf remove nmap
dnf history           # 트랜잭션 이력
```

`dnf remove`는 해당 패키지를 필요로 하는 다른 패키지까지 연쇄로 제거합니다. 실행 전에 `Removing:` 목록을 확인해야 합니다.

### 커널은 예외

보통 한 패키지는 한 버전만 설치되는데, 커널은 여러 버전이 동시에 설치됩니다.

새 커널은 부팅해봐야 되는지 알 수 있고, 안 되면 이전 커널로 돌아가야 하기 때문입니다.

```bash
dnf list kernel       # 설치된 커널 전부
uname -r              # 지금 돌고 있는 커널
```

### 그룹

```bash
dnf group list
dnf group info 'Development Tools'
dnf group install 'Development Tools'
```

```
정규 그룹    패키지 묶음
환경 그룹    정규 그룹의 묶음 (Server with GUI, Minimal Install)
```

패키지는 mandatory / default / optional로 표시되고, optional은 따로 지정하지 않으면 안 깔립니다.

---

## 6. 리포지토리 설정

### `dnf install` 할 때 벌어지는 일

```
/etc/yum.repos.d/*.repo 를 읽는다
   ↓ enabled=1 인 것만
창고의 패키지 목록표(메타데이터)를 받는다
   ↓
패키지를 찾고 의존성을 계산한다
   ↓
.rpm 을 내려받는다
   ↓
GPG 서명을 검사한다
   ↓
설치
```

`.repo` 파일은 이 흐름의 맨 앞에 쓰이는 주소록입니다.

### 파일 위치

```
/etc/yum.repos.d/*.repo    ← 리포지토리 정의. 이쪽을 씀
/etc/dnf/dnf.conf          ← dnf 자체 설정
```

둘 다 읽지만 `.repo` 쪽이 우선이고, Red Hat도 `.repo`를 쓰라고 권장합니다.

### 파일 내용 해부

Rocky 10의 실제 `/etc/yum.repos.d/rocky.repo` 입니다.

```ini
[baseos]                                          ← ① repo id
name=Rocky Linux $releasever - BaseOS             ← ② 사람이 읽는 이름
mirrorlist=https://mirrors.rockylinux.org/...     ← ③ 창고 주소
#baseurl=http://dl.rockylinux.org/...             ← ④ (주석 처리됨)
gpgcheck=1                                        ← ⑤
enabled=1                                         ← ⑥
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-Rocky-10   ← ⑦
```

**① repo id** — 명령어에서 쓰는 이름. 공백 없는 한 단어
**② name** — `dnf repolist` 출력에 보이는 설명일 뿐, 명령어에서는 안 씀

**③ mirrorlist / ④ baseurl** — 둘 중 하나만 있으면 됩니다

```
baseurl      주소 직접 지정. "이 URL에서 받아라"
mirrorlist   미러 목록 서버. "가까운 미러를 알아서 골라줘"
```

Rocky는 전 세계 미러가 있어서 mirrorlist가 기본입니다. 사내 리포지토리를 만들면 대개 baseurl을 씁니다.

`$releasever`(10), `$basearch`(aarch64)는 dnf가 채워넣는 변수입니다.

### `enabled=0`

같은 파일 안에 켜진 것과 꺼진 것이 같이 들어있습니다.

```ini
[baseos]              enabled=1    평소 쓰는 것
[baseos-debuginfo]    enabled=0    디버그 심볼. 평소엔 불필요
[baseos-source]       enabled=0    소스 코드
```

정의는 해두고 평소에는 사용하지 않는 리포지토리입니다. 필요할 때만 켭니다.

확인할 때 주의할 점이 있습니다.

```bash
dnf repolist        # 켜진 것만
dnf repolist all    # 꺼진 것까지 전부 + 상태
```

`repolist`만 실행하면 비활성 리포지토리는 표시되지 않습니다.

### 켜고 끄기 — 영구 vs 일시

```bash
# 영구. .repo 파일의 enabled 값을 실제로 고침
dnf config-manager --enable  baseos-debuginfo
dnf config-manager --disable baseos-debuginfo

# 일시. 이번 명령 실행 동안만. 파일은 그대로
dnf install pkg --enablerepo=baseos-debuginfo
dnf install pkg --disablerepo=epel
```

`config-manager --enable` 후에 파일을 다시 열어보면 `enabled=1`로 바뀌어 있습니다.

### 리포지토리 추가

```bash
dnf config-manager --add-repo="https://dl.fedoraproject.org/pub/epel/10/Everything/x86_64/"
```

`/etc/yum.repos.d/` 아래에 `.repo` 파일이 생깁니다. 직접 만들어도 됩니다.

```ini
[epel]
name=EPEL 10
baseurl=https://dl.fedoraproject.org/pub/epel/10/Everything/x86_64/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-10
```

### GPG 서명

받아온 `.rpm`이 진짜 그 배포처가 만든 것인지, 중간에 바꿔치기 되지 않았는지 검사합니다.

```
gpgcheck=1    검사해라
gpgkey=       검사에 쓸 공개키 위치
```

키가 없으면 설치가 **거부**됩니다. 그래서 외부 리포지토리를 쓸 때는 키를 먼저 가져옵니다.

```bash
rpm --import https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-10
```

설정 파일과 키를 통째로 RPM으로 배포하는 경우도 많습니다.

```bash
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-10.noarch.rpm
```

`--nogpgcheck`로 검사를 건너뛸 수 있지만, 위조/손상된 패키지가 들어올 수 있어서 실무에서는 쓰지 않습니다.

### `gpgcheck` 와 `repo_gpgcheck`

검사 대상이 둘로 나뉩니다.

```
gpgcheck=1        내려받은 .rpm 패키지 각각의 서명을 검증
repo_gpgcheck=1   리포지토리 메타데이터(repomd.xml)의 서명을 검증
```

`repo_gpgcheck`는 dnf 기본값이 `0`입니다. 패키지를 받기 전에 **패키지 목록 자체**가
바꿔치기되지 않았는지 확인하는 설정이라, 외부 리포지토리에서는 켜두기를 권장하는 곳이 많습니다.

메타데이터 검증은 파일 두 개를 대조하는 방식입니다.

```
repomd.xml       패키지 목록의 색인
repomd.xml.asc   그 색인에 대한 분리 서명
```

에러 메시지로 어느 단계에서 터졌는지 구분할 수 있습니다.

```
repomd.xml GPG signature verification error   ← 메타데이터 단계. 패키지는 아직 안 받음
Package XXX.rpm is not signed                 ← 패키지 단계
```

실제 사례는 `실무로-이어지는-지점.md` 참고.

---

## 정리

1. RPM 파일 이름은 `name-version-release.arch`. version은 원저자가, release는 패키저가 정한다
2. RPM 안에는 메타데이터 + cpio 아카이브 + 스크립트가 들어있다. 스크립트가 root로 돈다
3. `rpm`은 설치된 것 조회와 단일 파일 설치. 의존성은 못 푼다
4. `dnf`는 리포지토리에서 받아오고 의존성까지 해결한다
5. `rpm -qf`는 설치된 파일, `dnf provides`는 아직 안 깐 것
6. BaseOS는 OS 핵심(수명 고정), AppStream은 응용(수명 따로)
7. 리포지토리는 `/etc/yum.repos.d/*.repo`에 정의된다
8. `config-manager --enable`은 영구, `--enablerepo=`는 그 명령만

---

## 자주 묻는 질문

**Q. release 번호가 왜 필요한가요? version만 있으면 안 되나요?**
> 소스는 그대로인데 빌드 옵션을 바꾸거나 보안 패치만 얹어서 다시 말 때가 있습니다. 원저자 버전은 안 바뀌었으니 version은 그대로 두고 release를 올립니다. RHEL 보안 업데이트가 대부분 이 방식입니다.

**Q. noarch는 뭔가요?**
> 아키텍처를 타지 않는 패키지입니다. 셸 스크립트, 파이썬 코드, 문서, 폰트처럼 컴파일이 필요 없는 것들이요.

**Q. yum을 쓰면 안 되나요?**
> 동작은 똑같습니다. `/bin/yum`이 `dnf-3`로 가는 심볼릭 링크라서 같은 프로그램이 실행됩니다. Red Hat은 dnf를 쓰라고 권장합니다.

**Q. mirrorlist랑 baseurl 둘 다 쓰면요?**
> mirrorlist가 우선합니다. Rocky 기본 설정도 baseurl을 주석 처리해두고 mirrorlist를 씁니다.

**Q. `dnf update`랑 `dnf upgrade` 차이는요?**
> 지금은 같습니다. yum 시절에 구분이 있었는데 dnf에서는 upgrade의 별칭으로 통합됐습니다.

**Q. `dnf remove` 했더니 엉뚱한 게 같이 지워졌어요.**
> 그 패키지에 의존하는 패키지들이 연쇄로 지워집니다. 실행 전에 `Removing:` 목록을 확인해야 하고, 이미 지웠으면 `dnf history undo <id>`로 되돌릴 수 있습니다.

**Q. 인터넷이 안 되는 폐쇄망에서는요?**
> `baseurl=file:///경로` 로 로컬 디렉터리를 리포지토리로 쓸 수 있습니다. `createrepo` 명령으로 메타데이터를 만들어주면 됩니다.

---

실무에서 이 내용이 어떻게 나타나는지는 `실무로-이어지는-지점.md` 참고.
