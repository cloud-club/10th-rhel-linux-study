# Week 02 - Lab 01

## 실습 정보

- 발표자: 김기석
- 주제: 계정/그룹 관리 및 Sudo 권한
- 유형: 실습형

환경: Rocky Linux 10 (lima VM)

---

## 문제 1

`devteam` 그룹을 생성하세요.

### 수행 결과

```bash
groupadd devteam
grep devteam /etc/group
```

---

## 문제 2

`devuser` 사용자를 생성하세요. 홈 디렉터리가 있어야 하고, 로그인 셸은 `/bin/bash`, `devteam`은 보조 그룹이어야 합니다.

### 수행 결과

```bash
useradd -m -s /bin/bash -G devteam devuser
passwd devuser
```

문제 6에서 sudo를 써야 해서 비밀번호도 같이 설정했습니다.

---

## 문제 3

`adminteam` 그룹을 만들고, `devuser`가 기존 `devteam` 소속을 유지한 채 `adminteam`에도 속하도록 설정하세요.

### 수행 결과

```bash
groupadd adminteam
usermod -aG adminteam devuser
id devuser
```

`-a`를 빼면 보조 그룹이 교체되면서 `devteam`이 빠집니다.

![devuser 그룹 정보](./images/lab01-01.png)

---

## 문제 4

`devuser`의 비밀번호 정책을 최소 1일 / 최대 60일 / 만료 전 경고 7일로 설정하세요.

### 수행 결과

```bash
chage -m 1 -M 60 -W 7 devuser
chage -l devuser
```

![비밀번호 정책](./images/lab01-02.png)

---

## 문제 5

`adminteam` 그룹 사용자가 sudo를 통해 관리자 명령을 실행할 수 있도록 `/etc/sudoers.d/`에 별도 규칙을 구성하세요.

### 수행 결과

```bash
echo '%adminteam ALL=(ALL:ALL) ALL' > /etc/sudoers.d/adminteam
chmod 0440 /etc/sudoers.d/adminteam
visudo -cf /etc/sudoers.d/adminteam
```

그룹은 앞에 `%`를 붙입니다. 문법이 틀리면 sudo 자체가 안 되기 때문에 root 셸을 닫기 전에 `visudo -cf`로 확인했습니다.

---

## 문제 6

`devuser`로 새 로그인한 뒤, 그룹 소속과 sudo 권한이 실제로 적용되었는지 검증하세요.

### 수행 결과

```bash
su - devuser
whoami; sudo whoami; whoami
```

`devuser` → `root` → `devuser`로 나옵니다.

![sudo 실행 결과](./images/lab01-03.png)

---