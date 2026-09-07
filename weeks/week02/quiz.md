# 2주차 현장 개념 퀴즈

개념 점검 퀴즈 문제지입니다.
RHCSA Week 02 퀴즈

> 총 10문항 · 김기석 5문항 + 이주희 5문항 · 권장 5~10분

---

## PART 1. 김기석 - 계정/그룹 관리 및 Sudo 권한

### 1. Linux에서 사용자를 내부적으로 구분할 때 사용하는 고유 번호는 무엇입니까?

- a. UID
- b. GID
- c. PID
- d. umask

### 2. 기존 보조 그룹 소속을 유지하면서 사용자에게 새로운 보조 그룹을 추가하려고 합니다. 가장 적절한 명령 형태는 무엇입니까?

- a. `usermod -G newgroup user1`
- b. `usermod -aG newgroup user1`
- c. `groupmod -aG newgroup user1`
- d. `useradd -aG newgroup user1`

### 3. 다음 명령의 설정 의미로 올바른 것은 무엇입니까?

```bash
chage -m 1 -M 60 -W 7 user1
```

- a. 최소 1일, 최대 60일, 만료 7일 후 계정 삭제
- b. 최소 1일, 최대 60일, 만료 7일 전 경고
- c. 최소 60일, 최대 1일, 만료 7일 전 경고
- d. 1일 후 계정 잠금, 60일 후 삭제, 7회 로그인 허용

### 4. adminteam 그룹 전체에 sudo로 모든 관리자 명령 실행 권한을 주는 규칙으로 가장 적절한 것은 무엇입니까?

- a. `adminteam ALL=(ALL:ALL) ALL`
- b. `%adminteam ALL=(ALL:ALL) ALL`
- c. `@adminteam ALL=(ALL:ALL) ALL`
- d. `%adminteam ALL=(NONE) ALL`

### 5. 일반 사용자 devuser가 sudo 권한을 정상적으로 갖고 있을 때 다음 순서의 예상 결과는 무엇입니까?

```bash
whoami
sudo whoami
whoami
```

- a. devuser → devuser → devuser
- b. root → root → root
- c. devuser → root → devuser
- d. devuser → root → root

---

## PART 2. 이주희 - 파일 권한 및 umask

### 6. 권한 640을 올바르게 해석한 것은 무엇입니까?

- a. 소유자 rw-, 그룹 r--, 기타 ---
- b. 소유자 rwx, 그룹 r--, 기타 ---
- c. 소유자 r--, 그룹 rw-, 기타 ---
- d. 소유자 rw-, 그룹 rw-, 기타 r--

### 7. 파일의 기존 권한은 유지하면서 그룹에 쓰기 권한만 추가하려면 어떤 명령이 가장 적절합니까?

- a. `chmod g+w file1`
- b. `chmod g-w file1`
- c. `chmod u+w file1`
- d. `chmod o+w file1`

### 8. report.txt의 소유자를 alice, 소유 그룹을 devteam으로 한 번에 변경하는 명령은 무엇입니까?

- a. `chmod alice:devteam report.txt`
- b. `chgrp alice:devteam report.txt`
- c. `chown alice:devteam report.txt`
- d. `chown devteam:alice report.txt`

### 9. 현재 umask가 0027일 때 새 일반 파일의 기본 권한은 무엇입니까?

- a. 0666
- b. 0640
- c. 0750
- d. 0600

### 10. 현재 umask가 0022일 때 새 디렉터리의 기본 권한은 무엇입니까?

- a. 0644
- b. 0666
- c. 0755
- d. 0777
