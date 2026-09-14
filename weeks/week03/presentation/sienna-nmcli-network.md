# 네트워크 구성 관리

RH124 18장 (교재 569–617p)

---

## 1. NetworkManager

RHEL은 네트워크 설정을 **NetworkManager 서비스**가 관리합니다.
`nmcli`는 그 서비스에 명령을 보내는 CLI 도구입니다.

설정 파일을 직접 편집하는 대신 서비스를 통해 설정을 변경하는 구조입니다.

### 장치와 연결

```
장치(device)       실제 네트워크 인터페이스.  eth0, ens3, enp1s0
연결(connection)   그 장치에 씌울 설정 묶음.  "네트워크 프로필"이라고도 합니다
```

둘은 1:1이 아닙니다.

- 장치 하나에 연결 프로필을 여러 개 만들어둘 수 있습니다
- 단, 한 번에 활성화되는 건 하나뿐입니다

예를 들어 같은 무선 랜카드에 회사용 고정 IP 프로필과 집용 DHCP 프로필을 만들어두고
상황에 따라 전환합니다.

명령에 따라 받는 인자가 다릅니다.

```bash
nmcli con up   <연결이름>     # 연결 이름
nmcli dev dis  <장치이름>     # 장치 이름
```

---

## 2. RHEL 10에서 바뀐 것

```
ifcfg 파일과 /etc/sysconfig/network-scripts/  →  완전히 제거됨
```

RHEL 8/9 자료에는 `/etc/sysconfig/network-scripts/ifcfg-eth0`을 편집하라고 나오지만,
RHEL 10에는 해당 디렉터리가 없습니다.

지금은 keyfile 형식을 씁니다.

```
/etc/NetworkManager/system-connections/<연결이름>.nmconnection
```

INI 스타일의 키-값 파일입니다.

```ini
[connection]
id=lab0
type=dummy

[ipv4]
method=manual
address1=192.168.50.10/24,192.168.50.1
dns=1.1.1.1;8.8.8.8;
```

---

## 3. nmcli

### 조회

```bash
nmcli dev status          # 장치 목록과 상태
nmcli con show            # 연결 프로필 전체 (비활성 포함)
nmcli con show --active   # 활성 연결만
```

실제 출력 (lima VM)

```
DEVICE  TYPE      STATE                   CONNECTION
eth0    ethernet  connected               cloud-init eth0
lo      loopback  connected (externally)  lo
```

`lo`의 `connected (externally)`는 NetworkManager가 관리하지 않는 장치라는 뜻입니다.

연결 이름에 공백이 들어갈 수 있습니다(`cloud-init eth0`). 이런 건 따옴표로 감쌉니다.

### 명령어 축약

고유하게 식별되는 만큼만 줄여 쓸 수 있습니다.

```
nmcli connection modify   →  nmcli con mod
nmcli device disconnect   →  nmcli dev dis
```

### 연결 추가

```bash
# DHCP
nmcli con add ifname enp8s0 type ethernet

# 고정 IPv4
nmcli con add con-name enp9s0 type ethernet ifname enp9s0 \
  ipv4.method manual ipv4.addresses 192.168.0.5/24 ipv4.gateway 192.168.0.254

# IPv4 + IPv6
nmcli con add con-name enp10s0 type ethernet ifname enp10s0 \
  ipv6.method manual ipv6.addresses '2001:db8:0:1::c000:207/64' \
  ipv6.gateway '2001:db8:0:1::1' \
  ipv4.method manual ipv4.addresses '192.0.2.7/24' ipv4.gateway '192.0.2.1'
```

- `con-name`을 생략하면 `ethernet-enp8s0` 처럼 자동으로 붙습니다
- 같은 이름의 연결이 이미 있으면 실패합니다

### 수정

```bash
nmcli con mod enp8s0 ipv4.addresses 192.0.2.2/24 ipv4.gateway 192.0.2.254 \
  connection.autoconnect yes
```

### ipv4.method — auto 와 manual

DHCP 연결을 고정 IP로 바꿀 때 주소만 넣으면 안 됩니다.

```bash
# 이것만 하면 연결이 불완전해집니다
nmcli con mod enp1s0 ipv4.addresses 192.0.2.10/24

# method까지 바꿔야 합니다
nmcli con mod enp1s0 ipv4.method manual ipv4.addresses 192.0.2.10/24
```

```
ipv4.method auto     DHCP
ipv4.method manual   고정
```

method가 `auto`인 채로 고정 주소를 넣으면, DHCP로도 주소를 받고 고정 주소도 갖는
상태가 되거나 연결이 불완전하게 붙습니다.

### 값 목록 수정 — `+` / `-`

값이 여러 개인 설정은 기호로 더하고 뺍니다. 기호가 없으면 통째로 교체됩니다.

```bash
nmcli con mod enp8s0 +ipv4.dns 1.1.1.1     # 추가
nmcli con mod enp8s0 -ipv4.dns 1.1.1.1     # 제거
nmcli con mod enp8s0  ipv4.dns 1.1.1.1     # 기존 DNS 전부 교체
```

2주차 `usermod -aG`에서 `-a`를 빼면 기존 그룹이 교체되던 것과 같은 구조입니다.

### 확인

```bash
nmcli -p con show enp8s0                              # 읽기 좋게
nmcli -f ipv4.method,ipv4.addresses con show enp8s0   # 특정 필드만
```

출력에서 소문자는 관리자가 바꿀 수 있는 정적 속성, 대문자는 런타임 값입니다.

### 적용 / 끊기 / 삭제

```bash
nmcli con up enp8s0       # 활성화. 수정한 설정은 up 해야 반영됩니다
nmcli dev dis enp8s0      # 장치 연결 끊기
nmcli con del enp8s0      # 프로필 삭제
nmcli con reload          # 파일을 직접 고쳤을 때 다시 읽기
```

`nmcli con mod`는 NetworkManager와 직접 통신해서 파일에 바로 씁니다.
반면 파일을 손으로 편집했을 때는 `con reload`를 해야 반영됩니다.

파일을 단계적으로 만들어두고 준비되면 한 번에 로드하는 용도로 씁니다.

### 권한

```bash
nmcli gen permissions
```

```
root                        전부 yes
콘솔에 직접 로그인한 일반 사용자   대부분 허용
SSH로 붙은 일반 사용자         거의 없음  → root로 전환 필요
```

SSH로 붙어서 `sudo` 없이 `nmcli con add`를 하면 `Insufficient privileges`가 납니다.
NetworkManager가 콘솔 앞에 앉은 사용자를 노트북 주인으로 보고 권한을 더 주는 구조입니다.

---

## 4. hostname과 FQDN

### FQDN

**FQDN(Fully Qualified Domain Name, 정규화된 도메인 이름)** 은 네트워크에서 호스트를
고유하게 식별하는 전체 이름입니다.

```
web1.dev.example.com.
────  ─── ─────── ─── ─
 ①     ②     ③     ④  ⑤

① 호스트 이름
② 서브도메인
③ 도메인
④ 최상위 도메인(TLD)
⑤ 루트 도메인 — 보통 생략하지만 이게 진짜 끝입니다
```

끝의 점(`.`)이 루트 도메인입니다. 이게 있으면 더 이상 뒤에 붙일 게 없다는 뜻이 됩니다.
`dig` 출력에 `example.com.` 처럼 점이 붙어 나오는 이유입니다.

점이 없는 이름은 **짧은 이름(short name)** 이고, 시스템이 `search` 도메인을 붙여서
FQDN으로 만들어 조회합니다.

```
short name:  web1
search:      example.com
   ↓
실제 조회:    web1.example.com.
```

### hostname 확인과 변경

```bash
hostname                                # 현재 이름
hostname web1.example.com               # 임시 변경. 재부팅하면 원복
hostnamectl hostname db1.example.com    # 영구 변경
cat /etc/hostname
```

`hostname` 명령만 쓰면 메모리에만 반영됩니다.
영구 설정은 `/etc/hostname` 파일이고, `hostnamectl`이 그 파일을 대신 고쳐줍니다.

파일을 직접 편집해도 결과는 같습니다.

### hostname 세 종류

`hostnamectl`을 그냥 치면 세 가지가 나옵니다.

```bash
hostnamectl
```

```
   Static hostname: db1.example.com       ← /etc/hostname 에 저장. 영구
Transient hostname: host.example.com      ← 커널이 들고 있는 값. 재부팅하면 사라짐
   Pretty hostname: 성윤의 서버            ← 사람이 읽는 설명. 한글/공백 가능
```

```
Static      우리가 보통 말하는 hostname. 파일에 저장됩니다
Transient   DHCP나 역방향 DNS로 동적으로 정해지는 이름
Pretty      UI 표시용. 네트워크에서는 안 씁니다
```

`/etc/hostname`이 비어 있거나 없으면, 시스템은 인터페이스에 붙은 IP의
역방향 DNS 조회로 Transient hostname을 자동으로 정합니다.

```
   Static hostname: (unset)
Transient hostname: host.example.com
```

### 프롬프트

hostname을 바꾼 뒤에도 프롬프트에는 옛 이름이 남습니다.
셸이 시작할 때 읽은 환경 변수를 쓰기 때문입니다. 새 터미널을 열면 바뀝니다.

---

## 5. 이름 확인

### 순서

```
/etc/nsswitch.conf   ←  조회 순서를 정하는 파일
      ↓
/etc/hosts           ←  로컬 매핑. 여기서 찾으면 끝
      ↓
/etc/resolv.conf     ←  못 찾으면 DNS 서버에 질의
```

`/etc/nsswitch.conf`의 `hosts:` 줄이 순서를 정합니다. 기본은 `files` 먼저입니다.

### /etc/hosts

```
127.0.0.1   localhost localhost.localdomain localhost4
::1         localhost localhost.localdomain localhost6
172.25.254.254   classroom.example.com classroom
```

`IP  FQDN  짧은이름` 순으로 적습니다.

### /etc/resolv.conf

```
# Generated by NetworkManager
search example.com
nameserver 172.25.250.100
```

```
search      짧은 이름 뒤에 붙일 도메인 목록
nameserver  질의할 DNS 서버. 최대 3개 (앞의 것이 죽으면 다음으로)
```

이 파일은 직접 고쳐도 유지되지 않습니다. 맨 위 주석이 그 뜻입니다.
NetworkManager가 연결 프로필의 DNS 설정을 보고 다시 씁니다.

수정은 nmcli로 합니다.

```bash
nmcli con mod 'cloud-init eth0' +ipv4.dns 1.1.1.1
nmcli con up 'cloud-init eth0'
```

DHCP가 주는 DNS를 무시하려면:

```bash
nmcli con mod 'cloud-init eth0' ipv4.ignore-auto-dns yes
```

### getent 와 host / dig 의 차이

```bash
getent hosts node2.example.com    # 나옵니다
ping -c1 node2.example.com        # 나옵니다
host node2.example.com            # 못 찾습니다
dig  node2.example.com            # 못 찾습니다
```

```
getent, ping, curl   /etc/nsswitch.conf 순서를 따름 → /etc/hosts 를 봅니다
host, dig            DNS 서버에 직접 질의하는 진단 도구 → /etc/hosts 를 안 봅니다
```

"ping은 되는데 dig는 안 된다"는 상황이 여기서 나옵니다.
시스템의 다른 프로그램이 실제로 어떻게 해석하는지 보려면 `getent hosts`를 씁니다.

`host`와 `dig`는 `bind-utils` 패키지에 들어있습니다. 기본 설치에는 없을 수 있습니다.

```bash
dnf install bind-utils
```

---

## 정리

1. NetworkManager가 네트워크를 관리하고 nmcli는 그 서비스에 명령을 보냅니다
2. 장치와 연결은 1:1이 아닙니다. `con up`은 연결 이름, `dev dis`는 장치 이름
3. RHEL 10에서 ifcfg와 network-scripts는 사라졌습니다. 이제 `.nmconnection` keyfile
4. DHCP를 고정으로 바꿀 때 `ipv4.method manual`을 함께 지정해야 합니다
5. `+`/`-` 없이 값을 주면 기존 목록이 통째로 교체됩니다
6. `hostname`은 임시, `hostnamectl`은 영구(`/etc/hostname`)
7. FQDN은 끝의 점까지가 전체 이름이고, 짧은 이름은 `search`로 완성됩니다
8. `host`/`dig`는 `/etc/hosts`를 안 봅니다. `getent hosts`를 써야 합니다

---

## 자주 묻는 질문

**Q. 연결 이름과 장치 이름을 꼭 다르게 해야 하나요?**
> 같아도 됩니다. `con-name`을 생략하면 `ethernet-eth0` 처럼 장치 이름 기반으로 자동 생성됩니다. 다르게 두는 건 한 장치에 여러 프로필을 만들 때 구분하기 위해서입니다.

**Q. `nmcli con up`과 `nmcli dev connect` 차이는요?**
> `con up`은 특정 프로필을 활성화하고, `dev connect`는 그 장치에 맞는 프로필을 알아서 골라 올립니다. 프로필이 하나뿐이면 결과가 같습니다.

**Q. `ip addr`로도 IP를 설정할 수 있지 않나요?**
> 됩니다. 다만 `ip addr add`는 커널에 직접 넣는 거라 재부팅하면 사라집니다. 영구 설정은 NetworkManager 프로필로 해야 합니다. `ip` 명령은 현재 상태를 보거나 임시로 테스트할 때 씁니다.

**Q. `/etc/resolv.conf`를 직접 고치면 안 되나요?**
> NetworkManager가 다시 써버립니다. 굳이 직접 관리하려면 연결 프로필에 `ipv4.ignore-auto-dns yes`를 주거나 NetworkManager의 dns 관리를 꺼야 하는데, 일반적인 방법은 아닙니다.

**Q. FQDN 끝의 점은 왜 붙이나요?**
> 그게 루트 도메인입니다. 점이 있으면 완결된 이름이니 `search` 도메인을 붙이지 말라는 뜻이 됩니다. 없으면 짧은 이름으로 보고 `search` 목록을 붙여가며 조회를 시도합니다.

**Q. Transient hostname은 언제 쓰이나요?**
> `/etc/hostname`이 비어 있을 때 DHCP나 역방향 DNS로 자동으로 정해지는 이름입니다. 클라우드 인스턴스에서 흔히 보입니다. 영구 이름을 설정하면 Static이 우선합니다.

---

실무에서 이 내용이 어떻게 나타나는지는 `실무로-이어지는-지점.md` 참고.
