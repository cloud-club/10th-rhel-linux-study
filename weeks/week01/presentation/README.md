# 🐧 Week 01. Linux & RHEL Introduction

**Cloud Club 10기 · RHCSA Study**  
*리눅스 기초 OT · 터미널 · 파일 시스템 · 기본 명령어*

| 📅 Week | 🎯 Focus | 🧪 Practice Goal |
| :---: | :--- | :--- |
| **01** | Linux 기본기 (터미널 & 파일 시스템) | 직접 입력하며 명령어로 시스템을 다루는 감각 익히기 |

---

Cloud Club 10기 RHCSA Study · Linux 기초 OT 자료  
리눅스를 처음 접해도 앞으로의 RHCSA 스터디를 따라갈 수 있도록 터미널 · 파일 시스템 · 경로 · 기본 명령어에 익숙해지는 것을 목표로 합니다.  
이번 주는 시험 내용을 깊게 파기보다, "리눅스에서 명령어로 시스템을 다루는 감각"을 익히는 데 집중합니다.

---

## 1. Linux를 공부하는 이유

리눅스를 공부하다 보면 GUI보다 CLI(터미널)를 많이 사용하게 됩니다. 터미널에서는 명령어를 이용해 작업을 빠르게 수행할 수 있고, 같은 작업을 다시 실행하거나 여러 명령어를 조합해 자동화하기도 쉽습니다.

```bash
$ pwd
$ ls
$ cd /etc
```

결국 리눅스를 익힌다는 것은 **"명령어를 입력해서 시스템을 직접 다루는 것"**에 익숙해지는 과정이라고 생각하면 됩니다.

## 2. Linux에서 가장 먼저 익숙해져야 할 것

> 📌 **핵심 흐름**
>
> 위치 확인 → 파일 목록 확인 → 이동 → 생성/복사/이동/삭제 → 내용 확인 → 검색/조합

### 2.1 현재 위치 확인하기 - `pwd`

- 📌 **핵심**: 리눅스 작업을 시작할 때는 항상 현재 위치를 확인하는 습관을 들입니다.

`pwd`는 **Print Working Directory**의 약자입니다. 현재 내가 어느 디렉터리에 있는지 확인할 수 있습니다.

```bash
$ pwd
/home/user
```

처음 리눅스를 사용할 때는 항상 **"내가 지금 어디에 있지?"**를 확인하는 습관을 들이는 것이 좋습니다.

### 2.2 디렉터리 확인하기 - `ls`

- 📌 **핵심 명령어**: `ls` / `ls -alh`

현재 디렉터리에 어떤 파일과 디렉터리가 있는지 확인합니다.

```bash
$ ls
$ ls -alh
```

| **옵션** | **의미** |
| :---: | :--- |
| **`-a`** | 숨김 파일 포함 |
| **`-l`** | 자세한 정보 표시 |
| **`-h`** | 사람이 읽기 쉬운 크기로 표시 |

> 이름이 `.`으로 시작하는 파일은 기본 `ls`에서는 보이지 않습니다.

## 3. 디렉터리 이동하기 - `cd`

`cd`는 **Change Directory**의 약자입니다.

```bash
$ cd /etc
$ cd ~
$ cd ..
```

현재 위치를 확인하면서 이동해 보면 경로 개념을 익히기 쉽습니다.

```bash
$ pwd
$ cd ..
$ pwd
```

## 4. 절대경로와 상대경로

> ⚠️ **중요**: RHCSA 실습에서 경로를 잘못 잡으면 명령어 자체가 맞아도 작업이 실패할 수 있습니다.

| **표현** | **의미** |
| :--- | :--- |
| **`/`** | 루트 디렉터리 |
| **`.`** 또는 **`./`** | 현재 디렉터리 |
| **`..`** 또는 **`../`** | 상위 디렉터리 |
| **`~`** | 현재 사용자의 홈 디렉터리 |

- **절대경로**: 루트(`/`)부터 시작하는 전체 경로입니다.
  - `/home/user/test/file.txt`
- **상대경로**: 현재 위치를 기준으로 표현하는 경로입니다.
  - `./file.txt`
  - `../file.txt`

예를 들어 현재 위치가 `/home/user/test`라면 `cd ..`는 `/home/user`로 이동합니다.

## 5. 디렉터리 만들기 - `mkdir`

> 💡 **Tip:** 명령어 이름의 의미를 함께 기억하면 처음 보는 명령어도 훨씬 쉽게 익힐 수 있습니다.

`mkdir`은 **Make Directory**의 약자입니다.

```bash
$ mkdir test
$ mkdir alpha beta gamma
```

## 6. 디렉터리 구조 확인하기 - `tree`

`tree`는 디렉터리 구조를 트리 형태로 보여줍니다.

```bash
$ tree
$ tree -d
$ tree -a
$ tree -L 1
```

## 7. 파일과 디렉터리 다루기

🔑 **기억할 것**: `cp`는 복사, `mv`는 이동/이름 변경, `rm`은 삭제입니다.

| **명령어** | **역할** | **대표 예시** |
| :---: | :--- | :--- |
| **`cp`** | 복사 | `cp file.txt backup.txt` |
| **`mv`** | 이동 / 이름 변경 | `mv old.txt new.txt` |
| **`rm`** | 삭제 | `rm file.txt` |

### 7.1 복사 - `cp`

`cp`는 **copy**입니다.

```bash
$ cp file.txt backup.txt
$ cp -r directory1 directory2
```

- `-r`은 디렉터리 내부까지 재귀적으로 복사할 때 사용합니다.

### 7.2 이동 및 이름 변경 - `mv`

`mv`는 **move**입니다. 파일이나 디렉터리를 이동할 때뿐 아니라 이름을 바꿀 때도 사용합니다.

```bash
$ mv file.txt /tmp
$ mv old.txt new.txt
```

### 7.3 삭제 - `rm`

> 🚨 **주의**: `rm -rf`는 매우 강력한 삭제 명령입니다. 실행 전에 `pwd`와 `ls`로 대상을 반드시 확인합니다.

```bash
$ rm file.txt
$ rm -r directory
$ rm -rf directory
```

## 8. 파일 내용 다루기

- **`cat`**: 짧은 파일 빠르게 확인
- **`less`**: 긴 파일 탐색
- **`vi`**: 터미널에서 파일 수정

### 8.1 `cat`

텍스트 파일의 내용을 바로 확인합니다.

```bash
$ cat file.txt
```

### 8.2 `less`

내용이 많은 파일을 읽을 때 사용합니다.

```bash
$ less file.txt
```

- `Enter` / `Space`로 이동하고 `q`로 종료합니다.

### 8.3 `vi`

터미널에서 파일을 직접 수정할 때 사용하는 기본 편집기입니다.

```bash
$ vi file.txt
```

`INSERT` 모드에서 내용을 입력하고, `ESC`로 `COMMAND` 모드로 돌아갑니다.

```text
i       # INSERT 모드
ESC     # COMMAND 모드
:wq     # 저장 후 종료
:q!     # 저장하지 않고 종료
```

## 9. 명령어 결과를 조합하기

💡 **Linux의 핵심 사용 방식**: 명령어를 하나씩만 쓰는 것이 아니라 **Pipe와 Redirection으로 연결**해서 사용합니다.

> 🔗 **명령어 조합의 기본**
>
> 명령어 A ──|──> 명령어 B (Pipe)
>
> 명령어 출력 ──> 파일 (`>` / `>>`)

### 9.1 Pipe - `|`

앞 명령어의 결과를 뒤 명령어로 전달합니다.

```bash
$ cat file.txt | grep hello
```

### 9.2 `grep`

특정 문자열을 검색할 때 사용합니다.

```bash
$ grep hello file.txt
$ grep -i hello file.txt
$ grep -B 2 hello file.txt
$ grep -A 2 hello file.txt
```

### 9.3 Redirection - `>`, `>>`

명령어의 결과를 파일에 저장할 수 있습니다. `>`는 기존 내용을 덮어쓰고, `>>`는 기존 내용 뒤에 추가합니다.

```bash
$ ls > result.txt
$ ls >> result.txt
$ cat result.txt
$ tree /etc | grep sh > tree_result.txt
```

## 10. 파일 찾기 - `find`

파일이나 디렉터리를 이름으로 검색할 수 있습니다.

```bash
$ find /etc -name nginx.conf
$ find . -name "*.txt"
```

## 11. 압축과 아카이브 - `tar`

`tar`는 여러 파일이나 디렉터리를 하나의 파일로 묶는 아카이브에 사용됩니다. `gzip`과 함께 사용하면 압축된 `.tar.gz` 형태로 만들 수 있습니다.

```bash
$ tar -cvf archive.tar directory/
$ tar -xvf archive.tar

$ tar -czvf archive.tar.gz directory/
$ tar -xzvf archive.tar.gz
```

## 12. 시스템 관리 명령어도 미리 살펴보기

🧭 **미리 보기**: 이번 주에는 가볍게 익히고, 뒤에서 프로세스/네트워크를 본격적으로 다룹니다.

```bash
$ ps
$ ps -al
$ kill PID
$ ifconfig
```

이번 주에 깊게 다루지는 않지만, 앞으로 프로세스와 네트워크를 공부하면서 다시 만나게 됩니다. 실제 RHEL 환경에서는 이후 주차에서 `ip`, `nmcli` 등을 사용합니다.

## 13. 이번 주에는 이것만 익숙해져도 충분합니다

> ✅ **완벽하게 외우는 것보다 직접 입력해보는 것이 중요합니다.**  
> 아래 명령어들은 이후 주차에서도 계속 등장합니다.

- **경로**: `pwd` `cd` `.` `..` `~` `/`
- **파일 / 디렉터리**: `ls` `mkdir` `tree` `cp` `mv` `rm`
- **파일 확인 / 수정**: `cat` `less` `vi`
- **검색 / 조합**: `grep` `find` `|` `>` `>>`
- **기타**: `tar` `ps` `kill`

## 14. 직접 해보기

> 🧪 **Mini Lab**
>
> **목표**: 직접 입력하면서 **경로 → 디렉터리 → 파일 → 검색 → 결과 저장** 흐름을 익혀봅니다.

```bash
$ mkdir linux-study
$ cd linux-study

$ mkdir alpha beta
$ tree

$ cd alpha
$ pwd

$ vi test.txt
```

`vi`에서 아래 내용 입력 후 `:wq`로 저장합니다.

```text
Hello Linux
RHCSA Study
Cloud Club 10th
```

```bash
$ cat test.txt
$ cat test.txt | grep Linux
$ cat test.txt > result.txt
$ cat result.txt

$ cd ..
$ tree -a
```

> 📌 **Case-sensitive:** 리눅스는 대소문자를 구분합니다. 위에서 만든 디렉터리가 `linux-study`라면 이동할 때도 정확히 `linux-study`로 입력해야 합니다.

### ⚡ Command Quick Reference

| **구분** | **명령어** | **용도** |
| :--- | :--- | :--- |
| 📍 **위치** | `pwd` | 현재 위치 확인 |
| 📂 **탐색** | `ls`, `cd`, `tree` | 목록 확인 / 이동 / 구조 확인 |
| 🏗️ **생성** | `mkdir` | 디렉터리 생성 |
| 📋 **복사** | `cp` | 파일/디렉터리 복사 |
| 🚚 **이동** | `mv` | 이동 / 이름 변경 |
| 🗑️ **삭제** | `rm` | 파일/디렉터리 삭제 |
| 👀 **확인** | `cat`, `less` | 파일 내용 확인 |
| ✏️ **수정** | `vi` | 파일 편집 |
| 🔎 **검색** | `grep`, `find` | 내용 / 파일 검색 |
| 🔗 **조합** | `\|`, `>`, `>>` | 명령어 연결 / 결과 저장 |
| 📦 **묶기** | `tar` | 아카이브 생성 / 해제 |
| ⚙️ **시스템** | `ps`, `kill` | 프로세스 확인 / 종료 |

## 15. 앞으로의 스터디를 위한 핵심

> 🚀 **Week 1 → RHCSA**
>
> 이번 주 기본기가 앞으로 7주 동안 계속 사용됩니다. 이번 주 내용을 모두 외우는 것이 목표는 아닙니다.

```text
현재 위치 확인
      ↓
파일 / 디렉터리 확인
      ↓
경로를 이해하고 이동
      ↓
파일 생성 / 복사 / 이동 / 삭제
      ↓
파일 내용 확인
      ↓
검색 / 결과 조합
```

Linux 기본 명령어에 익숙해질수록 이후 RHCSA 실습도 훨씬 수월해집니다.

앞으로는 이 기본기를 바탕으로 아래 내용을 하나씩 공부합니다.

- 사용자 / 그룹
- 권한
- 네트워크
- 패키지
- SSH
- 스토리지 / LVM
- Systemd
- 방화벽
- SELinux
- 로그
- 부팅
- Shell Script

---

## 📚 참고 자료

- 📘 `쉬운 리눅스 강의자료.pdf`
- 📕 Red Hat 공식 RHCSA(EX200) Exam Objectives

---

> 🐧 **Next Week**
>
> 사용자 / 그룹 · 권한 · `sudo` · `chmod` · `chown` · `umask`
