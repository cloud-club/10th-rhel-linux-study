# Week 03 - Lab 01

## 실습 정보

- 발표자: 김성윤
- 주제: 네트워크 구성 / 패키지 관리
- 유형: 답안 작성형

---

## 문제 1

RHEL 10에서 NetworkManager가 네트워크 연결 프로필을 저장하는 위치는 어디입니까?

### 답안

```text
b. /etc/NetworkManager/system-connections/
```

### 간단 코멘트

RHEL 10에서는 기존 `ifcfg` 방식 대신 NetworkManager의 keyfile 형식을 사용하며, 연결 프로필은 `/etc/NetworkManager/system-connections/`에 저장됩니다.

---

## 문제 2

DHCP로 동작하던 `enp1s0` 연결에 고정 IP를 지정했지만 연결이 정상적으로 붙지 않습니다. 빠진 설정은 무엇입니까?

### 답안

```text
b. ipv4.method manual
```

### 간단 코멘트

DHCP 방식에서 고정 IP 방식으로 변경하려면 IP 주소만 지정하는 것이 아니라 `ipv4.method manual`도 함께 설정해야 합니다.

---

## 문제 3

`enp1s0` 연결에 기존 DNS 서버 `8.8.8.8`을 유지하면서 `1.1.1.1`을 추가하려면 어떤 명령이 적절합니까?

### 답안

```bash
b nmcli con mod enp1s0 +ipv4.dns 1.1.1.1
```

### 간단 코멘트

`+ipv4.dns`를 사용하면 기존 DNS 목록을 유지하면서 새로운 DNS 서버를 추가할 수 있습니다. `+` 없이 설정하면 기존 값이 교체될 수 있습니다.

---

## 문제 4

`/etc/hosts`에 추가한 항목을 반영하지 않는 명령은 무엇입니까?

### 답안

```text
c. dig web.example.com
```

### 간단 코멘트

`getent`, `ping`, `curl` 등은 시스템의 이름 조회 순서를 따르지만, `dig`는 DNS 서버에 직접 질의하므로 `/etc/hosts`의 항목을 사용하지 않습니다.

---

## 문제 5

`epel` 리포지토리가 `enabled=0`일 때 설정 파일을 변경하지 않고 이번 설치에서만 사용하려면 무엇을 사용해야 합니까?

### 답안

```bash
b dnf install pkg --enablerepo=epel
```

### 간단 코멘트

`--enablerepo`는 해당 명령을 실행하는 동안에만 지정한 리포지토리를 활성화합니다. `.repo` 파일의 설정 자체는 변경하지 않습니다.

---

## 배운 점

- RHEL 10에서는 NetworkManager 연결 프로필을 `.nmconnection` 형식으로 관리합니다.
- DHCP에서 고정 IP로 변경할 때는 `ipv4.method manual` 설정이 필요합니다.
- `nmcli`에서 `+`를 사용하면 기존 목록을 유지하면서 값을 추가할 수 있습니다.
- `/etc/hosts`를 포함한 실제 시스템 이름 조회 확인에는 `getent hosts`를 활용할 수 있습니다.
- `--enablerepo`를 사용하면 비활성화된 리포지토리를 일시적으로 사용할 수 있습니다.