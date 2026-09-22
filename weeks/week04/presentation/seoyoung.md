# Linux 4주차 개념 정리

---

## 0. grep과 find의 역할

|  | grep | find |
| --- | --- | --- |
| 찾는 것 | 파일 **내용** | 파일 **자체** |
| 결과 | 매칭된 **줄(텍스트)** | 조건에 맞는 **경로 목록** |
| 질문 형태 | "이 내용이 어디 적혀 있지?" | "이 조건의 파일이 어디 있지?" |

```bash
find /var/log -name "*.log"   # .log로 끝나는 파일의 경로
grep "error" app.log          # app.log 안에서 error가 있는 줄
```

- find = "파일 조건을 안다, 경로를 모른다"
- grep = "내용을 안다, 위치를 모른다"

| 궁금한 것 | 도구 |
| --- | --- |
| "이 설정값이 어디 적혀 있지?" | grep |
| "sshd_config 파일이 어디 있지?" | find |
| "1GB 넘는 파일 누가 쌓아놨지?" | find |
| "어제 로그에 timeout 몇 번 떴지?" | grep |

---

## 1. 파일시스템 기초 — inode 구조

- `size`의 동작, 디렉토리 크기, 권한 해석, 하드/심볼릭 링크가 모두 이 구조에서 나온다.

### 파일의 3조각

```
파일명 ──▶ inode ──▶ 데이터 블록
(표의 한 줄)  (메타정보)   (실제 내용)
```

| 조각 | 담긴 것 | 위치 |
| --- | --- | --- |
| **파일명** | inode 번호만 | 디렉토리의 데이터 블록 |
| **inode** | 타입, 권한, 소유자, 크기, 타임스탬프, 링크 카운트, **데이터 블록 위치** | inode 테이블 |
| **데이터 블록** | 실제 내용 | 디스크 |

**inode에는 파일명이 없다.** 그래서 방향이 한쪽이다.

### 디렉토리도 파일의 일종

inode 구조는 일반 파일과 **완전히 동일**하다. 차이는 데이터 블록의 내용물뿐이다.

|  | 일반 파일 | 디렉토리 |
| --- | --- | --- |
| inode | 메타정보 (동일) | 메타정보 (동일) |
| 데이터 블록 | 텍스트, 바이너리 등 | **이름 ↔ inode 번호 매핑 목록** |

디렉토리는 아래와 같은 **표 한 장**이고, 그 표가 내용물이다.

| 이름 | inode 번호 |
| --- | --- |
| big.bin | 131075 |
| small.bin | 131080 |
| a | 131082 |

"폴더 안에 파일이 있다"는 착시다. 실제 구조는 포함이 아니라 **참조**다.

### 이 구조로 설명되는 것들

| 현상 | 이유 |
| --- | --- |
| `mv`가 큰 파일도 즉시 끝남 | 표의 이름만 바꿈, 데이터는 안 움직임 |
| 하드 링크가 용량을 안 먹음 | 데이터 블록 공유 |
| 하드 링크가 파일시스템을 못 넘음 | inode 번호는 파일시스템 내부에서만 유효 |
| `rm` 후에도 데이터가 남을 수 있음 | 링크 카운트가 0이 아니면 해제 안 됨 |
| 디렉토리 하드 링크 금지 | 표가 순환 참조하면 트리가 깨지고 무한 루프 |

`rm`은 **디렉토리 표에서 이름 한 줄을 삭제**한다. 링크 카운트가 0이 될 때 데이터가 해제된다. → 실행 중인 프로그램의 파일을 지워도 도는 이유. (2주차 `userdel` 실패도 같은 참조 관계 문제)

### 확인 명령

```bash
stat big.bin       # inode에 담긴 정보 전체
ls -li big.bin     # inode 번호 + 링크 카운트
ls -ld a           # 디렉토리 자체 정보
```

---

## 2. 담당 영역 구분

```
파일명 ──▶ inode ──▶ 데이터 블록
└─── find의 영역 ───┘   └ grep의 영역 ┘
```

|  | find | grep |
| --- | --- | --- |
| 보는 곳 | 파일명 + inode (**메타정보**) | 데이터 블록 (**내용**) |
| 파일을 여나 | **안 연다** | **연다** |
| 결과 | 조건에 맞는 경로 목록 | 매칭된 줄 |

### find는 이름만 보지 않는다

이름은 조건 중 하나이고, 나머지는 inode의 메타정보다.

| 조건 | 보는 정보 | 어디서 |
| --- | --- | --- |
| `-name` | 이름 | 디렉토리의 매핑 표 |
| `-size` | 크기 | inode |
| `-mtime` | 수정 시각 | inode |
| `-perm` | 권한 | inode |
| `-user` | 소유자 | inode |
| `-type` | 파일/디렉토리/링크 | inode |

이름 조건이 없어도 검색된다.

```bash
find /var/log -size +50M      # 크기로만
find /etc -mtime -1           # 날짜로만
find /usr/bin -perm /u+s      # 권한으로만
```

→ "이름 중에서 찾는다"기보다 **"파일의 신상정보로 찾는다"**.

### 성능 차이

find는 inode만 읽으므로 **5GB 파일이든 0바이트 파일이든 검사 비용이 같다.**
grep은 파일을 통째로 읽으므로 용량에 비례해 느려진다.

### `grep -r`

기본 grep은 디렉토리를 거부한다.

```bash
$ grep "Port" /etc/ssh/
grep: /etc/ssh/: Is a directory
```

- `r`은 **"디렉토리를 받으면 안을 뒤져라"** 는 허가증이다.

```bash
$ grep -r "Port" /etc/ssh/
/etc/ssh/sshd_config:#Port 22
/etc/ssh/ssh_config:#   Port 22
```

내부 동작:

1. 디렉토리의 **매핑 표**를 읽어 파일 목록을 얻는다
2. 각 파일을 **하나씩 열어서** 패턴을 찾는다
3. 하위 디렉토리가 있으면 반복

순회는 검사할 파일을 모으는 수단이고, 판단은 끝까지 데이터 블록에서 한다. find는 반대로 디렉토리를 다 뒤지면서 파일을 한 번도 열지 않는다.

### 조합

```bash
find /etc -name "*.conf" -exec grep -l "Port" {} +
```

1. find가 `.conf`만 골라낸다 → inode 검사, 빠름
2. 그 파일들만 grep이 열어본다 → 느린 작업 최소화

`grep -r "Port" /etc/`도 되지만 바이너리·이미지·로그까지 열어서 느리고 `Binary file matches` 노이즈가 섞인다.

| 단계 | 역할 |
| --- | --- |
| find | **어떤 파일을 볼지** 결정 |
| grep | **그 안에서 무엇을 찾을지** 결정 |

---

## 3. grep

### 입력 경로

grep은 검색 대상을 **파일 경로 인자로 받는 것이 기본**이다. 다른 명령의 출력을 검색하려면 **파이프로 stdin에 넘긴다.**

| 찾을 대상 | 형태 |
| --- | --- |
| 파일 | `grep "패턴" 파일` |
| 명령의 출력 | `명령 | grep "패턴"` |

```bash
# 파일 → 인자로
grep "min" file.txt
grep -v "^#" /etc/ssh/sshd_config

# 명령 출력 → 파이프 (줄 "파일"이 없으므로 필수)
ps aux | grep sshd
dmesg | grep -i error
systemctl status sshd | grep Active
dnf list installed | grep kernel
ip a | grep inet
```

### 주요 옵션

| 옵션 | 의미 |
| --- | --- |
| `-i` | 대소문자 무시 |
| `-v` | 매칭 안 되는 줄 (반전) |
| `-n` | 줄 번호 |
| `-r` | 디렉토리 재귀 |
| `-w` | 단어 단위 매칭 |
| `-c` | 매칭된 줄 개수 |
| `-l` | 파일명만 출력 |
| `-o` | 매칭된 부분만 출력 |
| `-I` | 바이너리 파일 무시 |
| `-E` | 확장 정규표현식 (ERE) |
| `-A n` / `-B n` / `-C n` | 뒤 / 앞 / 양쪽 n줄 함께 |
| `--include="*.conf"` | 재귀 시 파일명 패턴 제한 |

### 필터링 패턴

```bash
# 주석·빈줄 제거
grep -v "^#" /etc/ssh/sshd_config | grep -v "^$"

# 여러 키워드 OR
dmesg | grep -iE "error|fail|warn"

# 특정 건 제외
dmesg | grep -i error | grep -v "usb"

# 에러 + 맥락 3줄 (매칭된 줄 앞뒤 n줄 함께 출력)
dmesg | grep -i -C 3 error 
```

로그는 한 줄만으론 맥락을 모르므로 `-C`가 유용하다. `-v`를 파이프로 이어붙이는 것이 가장 자주 쓰는 필터링 형태다.

---

## 4. find

```bash
find [경로] [조건] [동작]
```

- 동작 생략 시 기본은 `print`
- **조건 나열 = AND**
- **기본이 재귀** (`ls`와 달리 하위 디렉토리까지)

### 조건 옵션

| 조건 | 의미 |
| --- | --- |
| `-name` / `-iname` | 이름 (대소문자 구분 / 무시) |
| `-type f / d / l` | 파일 / 디렉토리 / 심볼릭링크 |
| `-size +10M` | 크기 (올림 함정 주의) |
| `-mtime -7` / `-mmin -30` | 수정 시각 (일 / 분) |
| `-user yong` | 소유자 |
| `-perm /u+s` | SUID |
| `-empty` | 빈 파일 또는 빈 디렉토리 |
| `-inum 131075` | inode 번호 |

`m` = modify. **생성 시각이 아니라 수정 시각**이다. (리눅스 표준 도구로는 생성 시각을 보기 어렵다)

### ⚠️ `size` 올림 함정

find는 크기를 비교하기 전에 **단위로 나눠서 올림**한다. 비교 단계에서는 단위를 보지 않고 **숫자끼리만** 비교한다.

1. 파일 크기 ÷ 단위 → **올림** → 개수
2. 그 개수를 내가 쓴 숫자와 비교 (단위 무시)

4096바이트 디렉토리:

| 명령 | 1단계 (나눠서 올림) | 2단계 (숫자 비교) | 결과 |
| --- | --- | --- | --- |
| `-size -1M` | 4096 ÷ 1M = 0.0039 → **1** | 1 < **1** | 거짓 |
| `-size -100k` | 4096 ÷ 100k = 0.04 → **1** | 1 < **100** | 참 |

올림하는 이유: `-size`는 용량이 아니라 **"단위 몇 개가 필요한가"** 를 센다. 4096바이트를 담으려면 1M 통이 1개 필요하다. 통은 쪼갤 수 없다. (파일시스템이 블록 단위로 할당하는 현실에 맞춘 설계)

| 명령 | 통 크기 | 4096B는 몇 통 | 기준 통 개수 |
| --- | --- | --- | --- |
| `-size -1M` | 1MB | 1통 | 1통 미만 |
| `-size -100k` | 100KB | 1통 | 100통 미만 |
- -1M은 "1MB 통 1개 미만" = 통이 아예 필요 없는 것 = 0바이트.
- -100k는 "100KB 통 100개 미만" = 통 1개짜리는 여유롭게 통과.

→ **`size -1M`은 사실상 "0바이트만"**

### `exec`

```bash
find . -name "*.log" -exec ls -lh {} \;
```

| 조각 | 역할 |
| --- | --- |
| `-exec` | 찾은 결과에 명령 실행 |
| `ls -lh` | 실행할 명령 |
| `{}` | 찾은 파일 경로가 들어갈 자리 |
| `\;` | -exec 명령의 끝 표시 |

**"파일에서 ls를 실행"이 아니라 인자로 주는 것**이다.

```bash
# 실제 실행되는 형태
ls -lh ./a/app.log
ls -lh ./b/error.log
```

- `{}` — 파일 경로를 받을 위치 명시
- `\;` — `;`는 쉘의 명령 구분자라서 이스케이프하지 않으면 쉘이 먼저 가로챈다 (`';'`도 동일)

| 방식 | 예시 |
| --- | --- |
| **인자로 받음** | `rm a.log`, `ls a.log`, `tar -czvf x.gz a.log` |
| **stdin으로 받음** | `grep "x"`, `sort`, `wc -l` |

---

### 실무 명령

```bash
# 큰 로그 파일 (디스크 꽉 찼을 때)
sudo find /var/log -type f -size +50M 2>/dev/null
```

`/var/log`에는 root만 읽을 수 있는 파일이 많아 sudo가 필요하다.

```bash
# 최근 변경된 설정 파일 (뭐 건드렸나 추적)
sudo find /etc -type f -mtime -1 2>/dev/null
sudo find /etc -type f -mmin -30 2>/dev/null
```

설정 바꾸고 서비스가 죽었을 때 원인 추적용.

```bash
# SUID 실행파일 (1주차 권한 연결)
find /usr/bin -perm /u+s -type f 2>/dev/null
```

`passwd`, `sudo` 등이 보인다 → 일반 유저가 패스워드를 바꿀 수 있는 이유.

```bash
# 빈 파일 / 빈 디렉토리
find . -empty
find . -empty -type f
find . -empty -type d
```

- `size 0c` 대신 `empty`가 필요한 이유: **디렉토리는 비어 있어도 4096바이트**다. `empty`는 크기가 아니라 항목 수를 본다.

```bash
find . -empty -type f -exec rm {} +
find . -empty -type d -delete
```

- `delete`는 `exec rm`보다 빠르지만 위험하다. → **먼저 `delete` 없이 돌려 목록을 확인한다.**

### 공통 구조

```
find  [어디서]  [무슨 조건]  [뭘 할지]
```

| 예시 | 어디서 | 조건 | 동작 |
| --- | --- | --- | --- |
| 1 | `.` | `-name "*.log"` | `-exec ls -lh` |
| 2 | `/var/log` | `-type f -size +50M` | (출력) |
| 3 | `/etc` | `-type f -mtime -1` | (출력) |
| 4 | `.` | `-empty` | (출력) |

---

## 6. tar — 아카이브

**아카이브(묶기)와 압축(줄이기)은 다른 일이다.**

|  | 하는 일 | 도구 |
| --- | --- | --- |
| **아카이브** | 여러 파일을 하나로 묶기 | tar |
| **압축** | 크기 줄이기 | gzip, bzip2, xz |

```
dir/ ──tar──▶ backup.tar ──gzip──▶ backup.tar.gz
     (묶기)              (압축)
```

`.tar.gz`라는 이름이 이 순서의 기록이다. gzip이 원래 "파일 하나의 크기를 줄이는" 도구로 만들어져 여러 파일 개념이 없기 때문에 나뉘어 있다.

### 기본 옵션

| 옵션 | 의미 | 역할 |
| --- | --- | --- |
| `c` | create | 묶기 |
| `x` | extract | 풀기 |
| `t` | list | 목록만 |
| `v` | verbose | 출력 자세히 |
| `f` | file | **파일명 지정** |

`c`, `x`, `t` 중 하나는 반드시 필요하다.

```bash
tar -cvf backup.tar dir/     # 묶기
tar -xvf backup.tar          # 풀기
tar -tvf backup.tar          # 목록 확인
```

### `f`는 항상 마지막

`f`는 **바로 다음 인자를 파일명으로 읽는다.**

```bash
tar -cvf backup.tar dir/     # 정상
tar -cfv backup.tar dir/     # 망함 — "v"를 파일명으로 읽음
```

두 번째는 `v`라는 이름의 tar를 만들려 하고 `backup.tar`를 묶을 대상으로 착각한다.

`f`가 따로 있는 이유: tar = **T**ape **AR**chive. 기본 출력이 테이프 드라이브였고 파일로 저장하려면 명시해야 했다.

### 압축까지 한 번에

| 옵션 | 도구 | 확장자 |
| --- | --- | --- |
| `z` | gzip | .tar.gz / .tgz |
| `j` | bzip2 | .tar.bz2 |
| `J` | xz | .tar.xz |

```bash
tar -czvf backup.tar.gz  dir/
tar -cjvf backup.tar.bz2 dir/
tar -cJvf backup.tar.xz  dir/
```

대소문자 주의: `j` = bzip2, `J` = xz.

tar가 압축을 직접 하지 않고 **해당 명령을 불러 파이프로 넘긴다.** 아래와 동일하다.

```bash
tar -cvf - dir/ | gzip > backup.tar.gz
```

- 는 "파일 대신 표준출력". `z`는 이걸 줄여 쓴 문법 설탕이다.

**풀 때는 압축 옵션 생략 가능** — 헤더를 보고 자동 감지한다. 묶을 때는 만들 형식을 알 수 없으므로 안 된다.
→ **`c`에는 z/j/J 필수, `x`에는 선택.**

### 추가 옵션

```bash
tar -xvf backup.tar.gz -C /tmp/restore       # 지정 위치에 풀기
tar -tf backup.tar.gz | head                 # 미리보기
tar -czvf backup.tar.gz --exclude="*.log" dir/
```

- `C`(change directory)를 안 쓰면 현재 위치에 파일이 쏟아진다. 최상위 디렉토리 없이 파일이 흩어진 아카이브("tar bomb")가 있으므로 남의 아카이브는 `t`로 먼저 확인하거나 `C`로 격리한다.

### find와 조합 — `T`

```bash
find /var/log -name "*.log" -mtime +30 | tar -czvf old-logs.tar.gz -T -
```

- `T` = files from. "묶을 파일 목록을 어디서 읽을지" 지정한다. tar는 원래 대상을 인자로 받으므로, 파일이 수백 개면 목록을 따로 넘겨야 한다.

```bash
tar -czvf backup.tar.gz -T list.txt    # 목록 파일에서
tar -czvf backup.tar.gz -T -           # 목록을 stdin에서
```

**파이프만으로 안 되는 이유**: tar는 stdin을 기본적으로 **"압축 해제할 아카이브 데이터"** 로 해석한다. `-T -`가 "stdin을 목록으로 읽어라"고 모드를 지정한다.

- 는 어느 옵션에 붙었냐로 대상이 달라진다.

| 표기 | 의미 |
| --- | --- |
| `-f -` | **아카이브**를 stdin/stdout으로 |
| `-T -` | **파일 목록**을 stdin으로 |

```bash
tar -czvf - dir/ | ssh server "cat > backup.tar.gz"   # 아카이브를 stdout으로
find . -name "*.log" | tar -czvf backup.tar.gz -T -   # 목록을 stdin으로
```

같은 결과를 내는 세 방법:

```bash
find ... | tar -czvf old.tar.gz -T -
find ... | xargs tar -czvf old.tar.gz
find ... -exec tar -czvf old.tar.gz {} +
```

- **`exec ... \;`만 안 된다.**

```bash
find . -name "*.log" -exec tar -czvf logs.tar.gz {} \;    # 망함
```

파일당 한 번 실행되어 tar가 매번 새로 만들므로 **마지막 파일만 남는다.**

공백·개선 문자가 든 파일명은 널 구분을 쓴다. `-T -`는 줄 단위로 읽기 때문이다.

```bash
find ... -print0 | tar -czvf old.tar.gz --null -T -
```

(xargs는 `-print0 | xargs -0`)

---

## 7. gzip / bzip2 / xz — 압축

### 단독 사용

```bash
gzip access.log          # access.log.gz 생성, 원본 삭제
gunzip access.log.gz     # 복원
```

**원본이 사라지는 게 tar와 다른 점이다.** tar는 원본을 남기지만 gzip은 압축본으로 **대체**한다.

```bash
gzip -k access.log                    # keep
gzip -c access.log > access.log.gz    # stdout으로 받기
```

해제 방법:

```bash
gunzip file.gz    /    gzip -d file.gz
bunzip2 file.bz2  /    bzip2 -d file.bz2
```

### 안 풀고 읽기

```bash
zcat access.log.gz | grep error
zgrep "error" access.log.gz
```

`/var/log`의 `*.log.1.gz` 같은 로테이션 로그를 볼 때 풀었다 지우는 과정이 없어진다. bzip2는 `bzcat`, `bzgrep`.

### 도구 비교

| 도구 | 압축률 | 속도 | 용도 |
| --- | --- | --- | --- |
| gzip | 보통 | 빠름 | 기본, 대부분의 경우 |
| bzip2 | 좋음 | 느림 | 요즘 xz에 밀려 덜 씀 |
| xz | 매우 좋음 | 매우 느림 | 배포용, 장기 보관 |
- **자주 만들고 자주 푼다** → gzip
- **한 번 만들고 여러 번 배포한다** → xz

```bash
gzip -1 file     # 빠르게, 덜 압축
gzip -9 file     # 느리게, 많이 압축
```

### 요약

| 목적 | 명령 |
| --- | --- |
| 묶고 압축 | `tar -czvf 결과.tar.gz 대상/` |
| 풀기 | `tar -xvf 파일.tar.gz` |
| 위치 지정해 풀기 | `tar -xvf 파일.tar.gz -C /경로` |
| 내용 확인 | `tar -tf 파일.tar.gz` |
| 파일 하나 압축 | `gzip 파일` (원본 사라짐) |
| 압축 파일 읽기 | `zcat`, `zgrep` |

---

## 8. Hard Link / Symbolic Link

### 8-1. 생성

```bash
ln    원본 링크      # 하드 링크
ln -s 원본 링크      # 심볼릭 링크 (s = symbolic)
```

옵션 하나 차이지만 만들어지는 것이 완전히 다르다.

### 8-2. 하드 링크 — 이름표 추가

`ln big.bin backup.bin`은 새 파일을 만들지 않는다. **디렉토리 표에 줄 하나가 추가되고 링크 카운트가 1 올라간다.**

```
디렉토리 표
├─ big.bin    ──▶ inode 131075 ──▶ 실제 데이터
└─ backup.bin ──▶ inode 131075 ──┘   (같은 곳)
```

- 둘은 **완전히 동등**하다. "원본/사본" 구분이 없고, 파일시스템은 어느 쪽이 먼저 생겼는지 모른다
- 한쪽으로 수정하면 다른 쪽에도 반영된다 (같은 파일이므로)
- **용량을 먹지 않는다.** 데이터 블록을 공유하므로 5MB 파일에 링크 10개를 걸어도 5MB

### 8-3. 심볼릭 링크 — 경로 문자열을 담은 파일

**별도 inode를 가진 새 파일**이고, 그 파일의 내용물이 "원본 경로 문자열"이다.

```
symlink ──▶ inode 131080 ──▶ 데이터: "big.bin"  (문자열)
                                    ↓ 이 경로를 다시 해석
                              big.bin ──▶ inode 131075 ──▶ 데이터
```

`echo "big.bin" > memo.txt`와 구조가 같다. 차이는 **inode의 타입 필드가 `l`** 이라서, 커널이 그 문자열을 "따라갈 경로"로 해석한다는 점이다. 타입이 `f`면 그냥 텍스트다.

그래서 크기가 경로 길이와 같다.

```bash
$ ls -l symlinklrwxrwxrwx 1 ... 7 symlink -> big.bin#                 ↑ "big.bin" = 7글자
```

5MB 파일을 가리켜도 링크 자체는 7바이트다. 복제가 아니라는 증거.

### 8-4. 하드 / 심볼릭 / cp 비교

|  | 새로 생기는 것 | 데이터 블록 |
| --- | --- | --- |
| **하드 링크** | 표의 줄 1개 | 공유 (안 만듦) |
| **심볼릭 링크** | 표의 줄 + **inode** + 데이터 블록 | 새로 만듦 (내용: 경로 문자열) |
| **복사 (`cp`)** | 표의 줄 + inode + 데이터 블록 | 새로 만듦 (내용: **원본 전체**) |

**접근 단계**

```
big.bin     : 표 → inode → 데이터                              (2단계)
backup.bin  : 표 → inode → 데이터                              (2단계, 위와 동일 경로)
symlink     : 표 → inode → "big.bin" → 표 → inode → 데이터      (경로 해석 한 번 더)
```

- 하드 링크는 원본과 **완전히 같은 경로**를 탄다. 성능 차이도 없고 구분도 안 된다.
- 심볼릭 링크는 문자열을 읽고 경로를 처음부터 다시 찾는 단계가 끼어든다. 이 단계 때문에 원본이 없으면 깨진다.

### 8-5. 원본 삭제 / 이동 시

`rm`이 하는 일은 두 가지뿐이다. (시스템 콜 이름은 실제로 `unlink()`)

1. 디렉토리 표에서 이름 한 줄을 지운다
2. 그 inode의 링크 카운트를 1 줄인다

**데이터 삭제 명령이 아니다.** 데이터 해제는 카운트가 0이 됐을 때의 부수 효과다.

- **하드 링크** — 카운트 2 → 1. 0이 아니므로 데이터 유지.
    
    ```
    rm 전:  big.bin ┐
                    ├─▶ inode 131075 ─▶ 데이터
            backup.bin ┘
    
    rm 후:  backup.bin ─▶ inode 131075 ─▶ 데이터    ← 정상 접근
    ```
    
    `backup.bin`은 여전히 완전한 파일이다. "파일은 지웠는데 데이터만 남은" 상태가 아니라, **애초에 그 데이터에 이름이 두 개 붙어 있었고 하나를 뗀 것.**
    
- **심볼릭 링크** — 링크와 그 내용("big.bin" 문자열)은 멀쩡하지만, 그 경로로 표를 찾았을 때 해당 줄이 없다.
    
    링크가 지워지는 게 아니라 **가리킬 대상만 없는 상태**(broken/dangling link)다. 실패 원인은 데이터 해제가 아니라 **경로 해석 실패**다. 
    
    이게 드러나는 경우:
    
    ```bash
    rm big.bin               # 데이터 영구 소멸
    cat symlink              # No such file (깨진 상태)
    
    echo "쓰레기" > big.bin   # 완전히 다른 파일
    cat symlink              # 쓰레기        ← 아무 의심 없이 따라감
    ```
    
    원본 데이터는 영구히 사라졌는데 링크는 정상 동작한다. 경로에 뭔가 있으면 그게 뭐든 따라간다.
    하드 링크는 inode를 붙잡고 있어서 이런 일이 없다. 같은 이름의 새 파일은 다른 inode다.
    

### 8-6. 차이 정리

| 항목 | Hard Link | Symbolic Link |
| --- | --- | --- |
| inode | 원본과 **동일** | **별개** |
| 실체 | 디렉토리 표의 한 줄 | 경로 문자열을 담은 파일 |
| 가리키는 대상 | **inode 번호** | **경로 문자열** |
| 원본 삭제 시 | 데이터 유지 | **링크 깨짐** |
| 원본 이동 시 | 무영향 | **링크 깨짐** |
| 파일시스템 경계 | 못 넘음 | 넘음 |
| 디렉토리 대상 | **불가** | 가능 |
| 없는 대상 지정 | 불가 | 가능 |
| 링크 카운트 | 증가시킴 | 안 건드림 |
| `ls -l` 첫 글자 | `-` (일반 파일과 동일) | `l` + `-> 경로` |
| 원본/사본 구분 | 없음 (동등) | 있음 (방향성) |

### 8-7. 하드 링크 제약의 원인

**세 제약 모두 "표에 적을 수 있는 게 inode 번호뿐"이라서 생긴다.**
디렉토리 표에는 `이름 → inode 번호` 두 칸밖에 없다.

**파일시스템 경계**

inode 번호는 그 파일시스템 안에서만 통하는 번호다. `/`의 131075번과 USB의 131075번은 무관한 다른 파일이다.

**디렉토리 금지**

기술적 불가가 아니라 **커널이 금지**한다. 디렉토리에 하드 링크를 허용하면 표가 순환 참조할 수 있다.

```
a/ 표에 b 항목 → b의 inode
b/ 표에 a 항목 → a의 inode      ← 무한 루프
```

`find`나 `ls -R`이 영원히 돌고, 링크 카운트 계산(`.` + `..` + 부모 항목)도 무너진다. 트리 구조라는 전제가 깨진다.

```bash
$ ln dir/ dirlinkln: dir/: hard link not allowed for directory
```

심볼릭 링크는 커널이 따라간 횟수를 셀 수 있어 허용된다. 40단계쯤 넘으면 `ELOOP` 에러로 멈춘다.

**없는 대상 지정**

하드 링크는 inode 번호를 알아야 만들 수 있는데, 없는 파일에는 inode가 없다.
심볼릭 링크는 문자열만 적으면 되므로 가능하다.

```bash
ln -s /not/yet/exist link     # 성공 (깨진 링크 상태로)
```

→ **inode 번호는 정확하지만 좁고, 경로 문자열은 느슨하지만 넓다.** 하드 링크의 제약은 정확함의 대가.

### 8-8. 실무 사례

**심볼릭 링크 — `systemctl enable`** 

```bash
systemctl enable sshdls -l /etc/systemd/system/multi-user.target.wants/# sshd.service -> /usr/lib/systemd/system/sshd.service
```

`enable`이 실제로 하는 일이 **심볼릭 링크 생성**이고, `disable`은 그 링크 삭제다.

| 위치 | 역할 |
| --- | --- |
| `/usr/lib/systemd/system/` | 패키지가 설치한 원본 (dnf 관리, 건드리면 안 됨) |
| `/etc/systemd/system/...wants/` | 관리자가 "쓴다"고 표시한 것 |

원본을 복사하거나 옮기지 않고 "이걸 쓰겠다"는 의사만 표현할 수 있다. 패키지를 업데이트해도 원본만 바뀌고 링크는 유지된다.
`/usr`와 `/etc`가 다른 파티션인 경우가 흔해 **하드 링크로는 불가능**하다.

**심볼릭 링크 — 타임존**

```bash
$ ls -l /etc/localtime/etc/localtime -> /usr/share/zoneinfo/Asia/Seoul
```

`/usr/share/zoneinfo/`에 전 세계 타임존 데이터가 있고, `/etc/localtime`은 그중 하나를 가리키는 포인터다.

```bash
timedatectl set-timezone America/New_Yorkls -l /etc/localtime        # 링크 대상만 바뀜
```

**심볼릭 링크 — 버전 전환 (alternatives)**

```bash
ls -l /etc/alternatives/ | head/usr/bin/python3 -> python3.11
```

버전 여러 개를 설치해두고 "현재 어느 걸 쓸지"를 링크로 표현한다. 버전을 올릴 때 링크 대상만 바꾸면 그걸 참조하는 모든 스크립트가 새 버전을 쓴다.

**심볼릭 링크 — 용량 부족 시 경로 우회**

```bash
mv /var/lib/docker /mnt/bigdisk/dockerln -s /mnt/bigdisk/docker /var/lib/docker
```

프로그램은 원래 경로를 계속 쓰는데 데이터는 다른 디스크에 있게 된다. **파일시스템을 넘을 수 있어서** 가능한 용법.

**하드 링크 — 증분 백업**

```bash
rsync -a --link-dest=/backup/day1 /data/ /backup/day2/
```

변경 안 된 파일은 `day1`의 하드 링크로 만든다. 100GB를 30일 보관해도 실제 용량은 100GB + 변경분. 각 날짜 디렉토리는 완전한 스냅샷처럼 보이지만 데이터를 공유한다. (Time Machine, rsnapshot이 이 방식)

**하드 링크 — 같은 프로그램 여러 이름**

```bash
ls -li /usr/bin/gzip /usr/bin/gunzip /usr/bin/zcat
```

같은 inode일 수 있다(배포판마다 다름). 프로그램이 자기가 어떤 이름으로 호출됐는지(`argv[0]`)를 보고 동작을 바꾼다. busybox는 하나의 바이너리가 수백 개 명령 역할을 한다.

### 8-10. 심볼릭 링크를 더 많이 쓰는 이유

**1. 제약이 적다**

실무에서 링크를 걸고 싶은 대상은 대부분 **디렉토리**거나 **다른 파티션/마운트**다. 하드 링크는 여기서 시작부터 탈락한다. (위 docker, systemd 예시 모두 하드 링크로는 불가능)

**2. 관계가 눈에 보인다**

bash

```bash
lrwxrwxrwx 1 ... python3 -> python3.11    # 관계가 즉시 보임-rw-r--r-- 2 ... file1                    # 하드 링크? 구분 불가
```

하드 링크는 일반 파일과 똑같이 표시된다. 카운트 2로 "뭔가 있다"는 건 알아도 **어디에 있는지는 `find -inum` 전수조사**가 필요하다. 운영 중인 시스템에서는 큰 단점.

**3. 방향성이 있다**

하드 링크는 "이 두 이름이 같은 파일"이라는 대등한 관계여서 어느 쪽이 진짜인지 개념이 없다.
심볼릭 링크는 "A는 B를 가리킨다"는 **방향 있는 관계**다. `python3 -> python3.11`처럼 "무엇이 현재 선택인가"를 표현하려면 방향이 필요하다.

**4. 실수의 영향이 작다**

하드 링크는 한쪽으로 수정하면 다른 쪽도 바뀐다. 백업이라 생각하고 하드 링크를 걸어뒀다가 원본 편집으로 "백업"까지 바뀌는 사고가 나기 쉽다.

**공통 패턴**: 실제 데이터는 한 곳에 두고, **선택이나 활성 상태만 링크로 표현**한다.

|  | 실제 데이터 | 링크가 표현하는 것 |
| --- | --- | --- |
| systemd | `/usr/lib/systemd/system/` | 어떤 서비스를 켤지 |
| 타임존 | `/usr/share/zoneinfo/` | 어느 타임존인지 |
| alternatives | 각 버전 실행파일 | 어느 버전을 쓸지 |

데이터를 복사하면 동기화 문제가, 옮기면 패키지 관리가 깨진다. 링크는 가리키기만 하므로 둘 다 피한다.