# 실습

작업 예약 (at, cron, systemd timer)

## 문제

### 1. 다음 중 매일 오후 2시 23분에 `/bin/echo hiya`를 실행하는 crontab 항목은 무엇입니까?
- a. `14 23 * * * /bin/echo hiya`
- b. `23 14 * * * /bin/echo hiya`
- c. `* * 23 14 * /bin/echo hiya`
- d. `23 2 * * * /bin/echo hiya`

### 2. 다음 중 at으로 예약해 둔 작업의 목록을 번호와 함께 확인하는 명령은 무엇입니까?
- a. `atrm`
- b. `atq`
- c. `at -c`
- d. `crontab -l`

### 3. root 사용자가 natasha 사용자의 cron 작업을 편집하려고 합니다. 올바른 명령은 무엇입니까?
- a. `crontab -e natasha`
- b. `crontab -u natasha -e`
- c. `cron -u natasha`
- d. `crontab -l -u natasha`

### 4. `/etc/systemd/system/` 에 `backup.service`와 `backup.timer`를 새로 만들었습니다. 타이머를 동작시키는 올바른 순서는 무엇입니까?
- a. `systemctl enable --now backup.service`
- b. `systemctl daemon-reload` > `systemctl enable --now backup.timer`
- c. `systemctl start backup.timer` > `systemctl daemon-reload`
- d. `crontab -e` 에 `backup.timer` 등록

### 5. `.timer` 파일의 `[Timer]` 섹션에 있는 `Persistent=true`는 어떤 역할을 합니까?
- a. 재부팅한 뒤에도 타이머가 자동으로 켜지게 합니다.
- b. 시스템이 꺼져 있어서 놓친 실행을 다음 부팅 직후 바로 실행합니다.
- c. 서비스가 비정상 종료되면 자동으로 다시 시작합니다.
- d. 실행 기록(로그)을 디스크에 영구 저장합니다.
---

# 정답

## 작업 예약 (at, cron, systemd timer) 실습 해설

### 1. 다음 중 매일 오후 2시 23분에 `/bin/echo hiya`를 실행하는 crontab 항목은 무엇입니까?

**정답:** **b.** `23 14 * * * /bin/echo hiya`

* **핵심 포인트:** crontab 한 줄은 왼쪽부터 **분 · 시 · 일 · 월 · 요일 · 명령** 순서입니다. 우리가 평소 시간을 읽는 순서(시 → 분)와 반대로 **분이 먼저** 온다는 점을 묻는 문제입니다. 오후 2시는 24시간제로 `14`입니다.
* **선지 설명:**
* **a.** `14 23 * * *`: 시와 분의 순서를 거꾸로 쓴 경우입니다. 매일 **밤 11시 14분**에 실행됩니다.
* **c.** `* * 23 14 *`: 일 칸에 23, 월 칸에 14가 들어갔습니다. 월은 1~12까지만 가능하므로 문법 오류입니다.
* **d.** `23 2 * * *`: 12시간제로 착각한 경우입니다. 매일 **새벽 2시 23분**에 실행됩니다.

---

### 2. 다음 중 at으로 예약해 둔 작업의 목록을 번호와 함께 확인하는 명령은 무엇입니까?

**정답:** **b.** `atq`

* **핵심 포인트:** `atq`는 **at + queue(줄)**, 즉 줄 서 있는 예약 목록을 보여 주는 명령입니다. 출력되는 작업 번호는 내용 확인(`at -c`)과 취소(`atrm`)에 사용됩니다.
* **선지 설명:**
* **a.** `atrm`: **at + remove**. 번호를 지정해 예약을 **취소**하는 명령입니다.
* **c.** `at -c`: 특정 번호의 예약 안에 들어 있는 **명령 내용**을 출력합니다. 목록을 보여 주지는 않습니다.
* **d.** `crontab -l`: at이 아니라 **cron 시간표**를 출력하는 명령입니다.

---

### 3. root 사용자가 natasha 사용자의 cron 작업을 편집하려고 합니다. 올바른 명령은 무엇입니까?

**정답:** **b.** `crontab -u natasha -e`

* **핵심 포인트:** 다른 사용자의 crontab을 다룰 때는 `-u 사용자이름` 옵션을 사용합니다. RHCSA 시험에서 "특정 사용자로 cron 작업을 구성하라"는 문제가 자주 출제되며, root로 접속한 상태에서 이 명령을 사용하는 것이 가장 확실합니다.
* **선지 설명:**
* **a.** `crontab -e natasha`: `-u` 옵션이 없어서 natasha를 사용자로 인식하지 못합니다. 원하는 동작을 하지 않습니다.
* **c.** `cron -u natasha`: `cron`(crond)은 시간표를 실행하는 **서비스(데몬)** 이름입니다. 시간표 편집은 `crontab` 명령으로 합니다.
* **d.** `crontab -l -u natasha`: natasha의 시간표를 **보기(list)** 만 합니다. 편집은 할 수 없지만, 작업 후 **확인용**으로 꼭 사용하는 명령입니다.

---

### 4. `/etc/systemd/system/` 에 `backup.service`와 `backup.timer`를 새로 만들었습니다. 타이머를 동작시키는 올바른 순서는 무엇입니까?

**정답:** **b.** `systemctl daemon-reload` > `systemctl enable --now backup.timer`

* **핵심 포인트:** 유닛 파일을 새로 만들거나 고친 뒤에는 `daemon-reload`로 systemd에게 **파일을 다시 읽게** 해야 합니다. 그리고 실제로 켜야 하는 것은 할 일(`.service`)이 아니라 **알람 시계(`.timer`)** 입니다. 타이머가 울리면 이름이 같은 서비스를 깨워 줍니다.
* **선지 설명:**
* **a.** `systemctl enable --now backup.service`: 서비스를 지금 한 번 실행할 뿐, 정해진 시간에 반복 실행되지 않습니다. 또한 `[Install]` 섹션이 없는 서비스는 enable 되지 않습니다.
* **c.** `systemctl start backup.timer` > `systemctl daemon-reload`: 순서가 반대입니다. 또한 `start`만 하면 재부팅 후에는 타이머가 꺼집니다.
* **d.** `crontab -e` 에 `backup.timer` 등록: cron과 systemd timer는 서로 다른 예약 방식입니다. crontab에는 실행할 명령을 적습니다.

---

### 5. `.timer` 파일의 `[Timer]` 섹션에 있는 `Persistent=true`는 어떤 역할을 합니까?

**정답:** **b.** 시스템이 꺼져 있어서 놓친 실행을 다음 부팅 직후 바로 실행합니다.

* **핵심 포인트:** systemd timer가 cron보다 "똑똑한" 이유입니다. 예를 들어 매일 새벽 2시 백업인데 그 시간에 컴퓨터가 꺼져 있었다면, `Persistent=true`가 있으면 다음에 켜지자마자 놓친 백업을 실행합니다.
* **선지 설명:**
* **a.** 재부팅 후 자동 시작: `[Install]` 섹션의 `WantedBy=timers.target`과 `systemctl enable`이 하는 일입니다.
* **c.** 비정상 종료 시 재시작: `.service` 파일의 `Restart=` 설정이 하는 일입니다.
* **d.** 로그 영구 저장: journald 설정과 관련된 내용으로, 타이머 옵션이 아닙니다.
---

## 요약
| 문제 | 핵심 |
| --- | --- |
| 1 | crontab 필드 순서 (분 → 시 → 일 → 월 → 요일) |
| 2 | at 관리 명령어 (`atq`, `at -c`, `atrm`) |
| 3 | 다른 사용자의 crontab 편집 (`crontab -u 사용자 -e`) |
| 4 | 타이머 적용 순서 (`daemon-reload` → `.timer` enable) |
| 5 | `Persistent=true` - 놓친 작업 실행 |
