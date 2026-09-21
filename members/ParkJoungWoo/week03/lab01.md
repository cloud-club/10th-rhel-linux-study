# Week 03 - Lab 01

## 실습 정보

- 발표자: 김성윤
- 주제: 
    - nmcli를 활용한 IPv4/IPv6 주소 및 Hostname 설정
    - DNF/RPM 리포지토리 구성 및 패키지 관리 (dnf, rpm)
- 유형: 퀴즈형

---

## 문제
# 실습 문제

네트워크 구성 / 패키지 관리


### 1. RHEL 10에서 NetworkManager가 네트워크 연결 프로필을 저장하는 위치는 어디입니까?

- a. `/etc/sysconfig/network-scripts/`
- b. `/etc/NetworkManager/system-connections/`
- c. `/etc/nmcli/profiles/`
- d. `/var/lib/NetworkManager/`

```
b
```
[rh9 공식문서](https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/9/html/configuring_and_managing_networking/assembly_networkmanager-connection-profiles-in-keyfile-format_configuring-and-managing-networking)
### 2. DHCP로 동작하던 `enp1s0` 연결에 아래 명령으로 고정 IP를 지정했습니다. 그런데 연결이 정상적으로 붙지 않습니다. 빠진 설정은 무엇입니까?

```bash
nmcli con mod enp1s0 ipv4.addresses 192.0.2.10/24 ipv4.gateway 192.0.2.1
nmcli con up enp1s0
```
- a. `connection.autoconnect yes`
- b. `ipv4.method manual`
- c. `ipv4.dns 8.8.8.8`
- d. `connection.interface-name enp1s0`
```
b
```
[rh9 공식문서 - 예 3.10. 대화형 편집기를 사용하여 정적 이더넷 연결 구성](https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/7/html/networking_guide/sec-adding_and_configuring_a_static_ethernet_connection_with_nmcli)

---

### 3. `enp1s0` 연결에 이미 DNS 서버 `8.8.8.8`이 설정되어 있습니다. 기존 설정을 유지하면서 `1.1.1.1`을 추가로 등록하려면 어떤 명령이 적절합니까?

- a. `nmcli con mod enp1s0 ipv4.dns 1.1.1.1`
- b. `nmcli con mod enp1s0 +ipv4.dns 1.1.1.1`
- c. `nmcli con add enp1s0 ipv4.dns 1.1.1.1`
- d. `nmcli con mod enp1s0 ipv4.dns-add 1.1.1.1`
```
b
man nmcli 중...
The + and - modifiers only have a real effect for properties that support them. These are for example multi-value (container) properties or flags like ipv4.dns, ip4, ipv4.addresses, bond.options, 802-1x.phase1-auth-flags etc.
```

---

### 4. `/etc/hosts`에 `192.168.1.50 web.example.com` 항목을 추가했습니다. 다음 중 이 항목을 **반영하지 않는** 명령은 무엇입니까?

- a. `ping web.example.com`
- b. `getent hosts web.example.com`
- c. `dig web.example.com`
- d. `curl http://web.example.com`
```
c
NSS가 적용되면 /etc/hosts를 조회함.
dig는 NSS에 영향을 받는 명령이 아님.
```
---

### 5. `epel` 리포지토리가 `enabled=0`으로 설정되어 있습니다. 설정 파일을 바꾸지 않고 **이번 설치 명령에서만** 이 리포지토리를 사용하려면 무엇을 써야 합니까?

- a. `dnf config-manager --enable epel` 실행 후 설치
- b. `dnf install pkg --enablerepo=epel`
- c. `dnf install pkg --repo epel --permanent`
- d. `.repo` 파일에서 `enabled=1`로 수정 후 설치
```
b
man dnf 발췌..
--enablerepo=<repoid>
    Temporarily enable additional repositories for the purpose
    of the current dnf command.  Accepts an id, a
    comma-separated list of ids, or a glob of ids. This option
    can be specified multiple times.
```
---

