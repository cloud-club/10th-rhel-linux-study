# 20261012

- at, crontab 작업 예약 및 Systemd Timer 유닛 활용
- 프로세스 우선순위 조정 (nice, renice) 및 tuned-adm 프로파일 적용

---

### 개요

> **작업 예약** : 사람이 직접 명령을 치지 않아도, 정해진 시간에 컴퓨터가 알아서 명령을 실행하게 맡기는 것

- 이점 1 : **사람이 없어도 된다** (새벽 백업, 주말 로그 정리)
- 이점 2 : **빼먹거나 틀리지 않는다** (반복 작업을 컴퓨터가 정확히 수행)

예약 도구는 세 가지

- **at** : 포스트잇 메모 → **딱 한 번만** 실행
- **cron** : 학교 시간표 → 정해진 때마다 **반복**
- **systemd timer** : 스마트 알람 → 반복 + **놓친 일 챙기기** + 실행 기록

---

### at - 한 번만 실행

- **atd** 서비스(예약 담당 직원)가 켜져 있어야 동작 → `systemctl enable --now atd`
- 예약 : `at now + 5 minutes` → `at>` 에 할 일 입력 → **Ctrl+D** 로 끝
- 한 줄 예약 : `echo "date > /tmp/test.txt" | at now + 1 hour`

#### 시간 표현

- `now + 5 minutes`, `now + 2 hours` : 지금부터 5분 / 2시간 뒤
- `17:30` : 오늘 오후 5시 30분 (이미 지났으면 내일)
- `noon`, `midnight`, `teatime` : 낮 12시, 밤 12시, 오후 4시
- `9am tomorrow`, `4pm + 3 days` : 내일 오전 9시, 3일 뒤 오후 4시

#### 관리

- `atq` : 예약 목록 (**at + queue**)
- `at -c 번호` : 예약 내용 보기 (**-c = cat**)
- `atrm 번호` : 예약 취소 (**at + remove**)

---

### cron - 정해진 때마다 반복

- **crond** 가 1분마다 시간표(**crontab**)를 확인해서 실행
- crontab 은 **사용자마다 따로** 있음

#### 명령어

- `crontab -e` : 내 시간표 고치기 (edit)
- `crontab -l` : 내 시간표 보기 (list)
- `crontab -r` : 내 시간표 **전부 삭제** (e 바로 옆이 r, 오타 주의!)
- `crontab -u natasha -e` : root 가 다른 사용자 시간표 고치기 (**시험 단골**)

#### 한 줄 읽는 법

**분 · 시 · 일 · 월 · 요일 · 명령** (작은 단위 → 큰 단위)

- 분 0-59 / 시 0-23 / 일 1-31 / 월 1-12 / 요일 0-7 (**0, 7 = 일요일**, 1 = 월요일)
- 예) `30 9 * * 1-5 /root/backup.sh` → 평일 오전 9시 30분마다 실행

#### 특수 기호

- `*` : 모든 값 (매번)
- `,` : 여러 개 → `1,15` = 1일과 15일
- `-` : 범위 → `1-5` = 월~금
- `/` : 간격 → `*/10` = 10분마다

#### 예시

- `*/5 * * * *` : 5분마다
- `0 9 * * 1-5` : 평일 오전 9시
- `30 2 1 * *` : 매달 1일 새벽 2시 30분
- `0 0 * * 0` : 매주 일요일 자정
- `23 14 * * *` : 매일 오후 2시 23분 → **분이 먼저!** (`14 23` 은 밤 11시 14분)

#### 시스템 crontab

- `/etc/crontab`, `/etc/cron.d/` : 칸이 **7개** (명령 앞에 **실행할 사용자** 이름 추가)
- `/etc/cron.daily/` 등 : 스크립트를 넣으면 매일 실행
- 특별한 지시가 없으면 `crontab -e` 사용 (저장 시 문법 검사 + 즉시 적용)

---

### systemd timer - 똑똑한 반복

- 이점 1 : **놓친 작업 챙기기** (꺼져 있던 동안 놓친 실행을 켜지자마자 실행)
- 이점 2 : **실행 기록** 이 남음 (`journalctl`)
- 이점 3 : `systemctl` 로 관리

> **.timer** = 언제? (알람 시계) / **.service** = 무엇을? (할 일)
> **이름이 같으면 짝꿍** : `backup.timer` 가 울리면 `backup.service` 실행

파일 위치 : `/etc/systemd/system/`

#### ① backup.service

- `[Service]` 섹션
  - `Type=oneshot` : 한 번 실행하고 끝나는 일
  - `ExecStart=/usr/local/bin/backup.sh` : 실행할 명령 (**전체 경로**)
- `[Install]` 섹션은 없어도 됨 (타이머가 깨워 주니까)

#### ② backup.timer

- `[Timer]` 섹션
  - `OnCalendar=*-*-* 02:00:00` : 매일 새벽 2시
  - `Persistent=true` : 꺼져 있어서 놓쳤으면 켜지자마자 실행
- `[Install]` 섹션
  - `WantedBy=timers.target` : 부팅할 때 타이머도 자동으로 켜기

#### OnCalendar 읽는 법

**요일 · 연-월-일 · 시:분:초** (요일 생략 가능, `*` = 매번, 평소 읽는 순서 그대로)

- `daily` : 매일 자정
- `hourly` : 매시간 정각
- `Mon..Fri *-*-* 09:00:00` : 평일 오전 9시
- `*-*-01 03:00:00` : 매달 1일 새벽 3시
- `*:0/15` : 15분마다

#### ③ 켜고 확인하기

1. `systemctl daemon-reload` : 새 파일 읽기 (**꼭!**)
2. `systemctl enable --now backup.timer` : **.service 가 아니라 .timer 를** 켠다
3. `systemctl list-timers` : 다음 실행 시각 확인
4. `systemd-analyze calendar "Mon..Fri 09:00"` : 시간 표현 미리 테스트
5. `journalctl -u backup.service` : 실행 기록 확인

#### 세 가지 비교

- **at** : 한 번 / 확인 `atq` / 취소 `atrm` / 서비스 `atd`
- **cron** : 반복 / 확인 `crontab -l` / 취소 `crontab -e` 에서 줄 삭제 / 서비스 `crond`
- **systemd timer** : 반복 + 놓친 일 챙기기 / 확인 `systemctl list-timers` / 취소 `systemctl disable --now 이름.timer`

---

### 프로세스 우선순위

> **프로세스** : 실행 중인 프로그램. 여러 프로세스가 CPU 를 나눠 쓴다

#### nice 값 - 급식 줄 비유

CPU = **배식대 하나**, 프로세스 = **줄 선 학생들**
nice = '착한' → 착할수록 **"먼저 드세요~"** 하고 양보

- 범위 **-20 ~ 19**, 기본값 **0**
- **-20** : 새치기 VIP (우선순위 최고) / **19** : 양보 천사 (우선순위 최저)
- **숫자가 작을수록 우선순위가 높다!** (거꾸로라 헷갈림 주의)

#### 누가 바꿀 수 있나

- 일반 사용자 : **올리기(양보)만** 가능
- root : 올리기, **내리기(새치기)** 모두 가능
- 한 번 올린 값을 되돌리는 것도 "내리기" → root 만 가능

#### nice / renice

- **nice** : 새로 **시작할 때**
  - `nice -n 10 명령` : nice 값 10으로 시작
  - `nice 명령` : `-n` 생략 시 기본 10
- **renice** : 이미 **실행 중일 때** (PID 로 지정)
  - `renice -n 15 -p 1234` : PID 1234 를 15로
  - `renice -n 19 -u student` : student 의 프로세스 전부

#### 확인

- `pgrep 이름` : PID 찾기
- `ps -o pid,ni,comm -p PID` : **NI** 칸 = nice 값
- `top` : **NI** = nice, **PR** = 20 + NI / `r` 키로 renice 가능

---

### tuned-adm - 컴퓨터 전체의 모드

> 스마트폰의 **절전 모드 · 게임 모드** 처럼, 하는 일에 맞춰 **프로파일(모드)** 을 고른다

- **tuned** : 프로파일에 맞게 CPU, 디스크, 네트워크, 전력 설정을 조정하는 서비스
- **tuned-adm** : 프로파일을 고르는 리모컨
- nice 가 **학생 한 명**의 순서라면, tuned 는 **급식실 전체**의 운영 방식

#### 명령어

- `systemctl enable --now tuned` : 서비스 켜기
- `tuned-adm list` : 프로파일 목록
- `tuned-adm recommend` : 추천 프로파일
- `tuned-adm profile 이름` : 프로파일 적용 (재부팅해도 유지)
- `tuned-adm active` : 현재 프로파일 확인


#### 주요 프로파일

- `balanced` : 성능과 절약의 균형 (기본 모드)
- `powersave` : 전기 절약 (절전 모드)
- `throughput-performance` : 많은 일을 빠르게 (스포츠 모드)
- `latency-performance` : 빠른 반응 (순발력 모드)
- `virtual-guest` : **가상 머신 안**의 리눅스 (**시험 환경!**)
- `virtual-host` : 가상 머신을 돌리는 호스트
