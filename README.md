# 🐧 Cloud Club 10기 - 리눅스 스터디

> **리눅스와 친해지고 RHCSA 자격증까지 따기!**  
> 리눅스를 제대로 이해하고 다루다 보면, RHCSA 자격증은 자연스럽게 따라옵니다!  
> 혼자 하면 어렵지만, 함께하면 끝까지 완주할 수 있습니다. 꼬인 에러를 함께 풀어가며 실무 관리 역량을 다집니다.

---

## 👥 Members (7명)

| <img src="https://github.com/user-attachments/assets/4ae8a4d3-995b-469d-827d-4d2f812d8fcb" width="150"><br>김기석 | <img src="https://github.com/user-attachments/assets/a8af0e3b-2d40-47c9-a8b7-006578b2dee6" width="150"><br>김보겸 | <img src="https://github.com/user-attachments/assets/fac54da4-c987-419e-84b6-8b18d957de6b" width="150"><br>김성윤 | <img src="https://github.com/user-attachments/assets/5e7a3c91-3529-4008-bd20-af08f7daaca6" width="150"><br>박시윤 |
| :---: | :---: | :---: | :---: |
| 스터디장 | 참여자 | 참여자 | 참여자 |
| [@kiseokkm](https://github.com/kiseokkm/) | [@k-kbk](https://github.com/k-kbk/) | [@sienna011022](https://github.com/sienna011022/) | [@salzzing](https://github.com/salzzing/) |

| <img src="https://github.com/user-attachments/assets/3f966cad-ce07-4aed-a6c3-7ae7917f2546" width="150"><br>박정우 | <img src="https://github.com/user-attachments/assets/299159c0-a4f3-4c3c-9072-7337f9410658" width="150"><br>이서영 | <img src="https://github.com/user-attachments/assets/dcd049c1-ea94-4287-a8ad-81bcd4207f24" width="150"><br>이주희 |
| :---: | :---: | :---: |
| 참여자 | 참여자 | 참여자 |
| [@ParkJoungWoo](https://github.com/ParkJoungWoo/) | [@7910trio](https://github.com/7910trio/) | [@BeanMouse](https://github.com/BeanMouse/) |

---

## 📂 저장소 구조 (Repository Structure)

```text
10th-rhel-linux-study/
├── .github/
│   └── pull_request_template.md    # 개인 실습 제출용 PR 템플릿
│
├── templates/                      # [공용] 개인 제출용 Markdown 템플릿
│   ├── README.md                   # 템플릿 사용 및 제출 방법 안내
│   ├── lab-answer-template.md      # 개념 / 명령어 답안 작성형 Lab
│   ├── lab-practice-template.md    # 실제 Linux 환경 실습형 Lab
│   └── quiz-template.md            # Quiz 답안 작성용 템플릿
│
├── weeks/                          # [공용] 회차별 스터디 세션 공유 자료
│   └── weekNN/                     # 주차별 폴더 (예: week01, week02)
│       ├── presentation/           # 당일 발표자 2인의 발표 자료
│       ├── labs/                   # 발표자가 출제한 Lab 문제
│       └── quiz.md                 # 세션 중 함께 진행하는 개념 Quiz
│
└── members/                        # [개인] 스터디원별 실습 결과 제출 공간
    └── <github-id>/                # 개인 GitHub ID 폴더 (예: kiseokkm)
        └── weekNN/
            ├── lab01.md            # Lab 01 결과
            ├── lab02.md            # Lab 02 결과
            ├── quiz.md             # Quiz 답안 (선택)
            └── images/             # 실습 캡처 이미지 (필요한 경우)
```

> 개인 제출 방법 및 템플릿 사용법은 [`templates/README.md`](./templates/README.md)를 참고해주세요.

---

## 🎯 스터디 목표 & 자료

- **목표**: Red Hat Certified System Administrator (EX200) 자격증 취득 및 RHEL 10 기반 실무 운영 역량 강화
- **주요 자료**:
  - Red Hat Official EX200 Exam Objectives (RHEL 10 기준 공식 출제 범위)
  - RHCSA 주요 출제 유형별 실전 시나리오 & 검증 가이드 (Node 1 / Node 2 기반)
  - 기술 블로그 & 공식 문서 큐레이션 자료

---

## 🛠️ 진행 방식 & 규칙

### ⏰ 스터디 시간

- **매주 월요일 21:00 ~ 22:30** (60분~90분)

### ⏱️ 세션 타임라인

- **20분 (10분 × 2명)**: 해당 주차 핵심 개념 발표
- **30분**: 발표자가 준비한 Lab 2개 풀이 및 현장 개념 Quiz
  - 발표자 2명이 각각 Lab 1개씩 출제
  - 실습 진행 상황 및 남은 시간에 따라 Quiz는 유연하게 진행
- **20분~**: 라이브 Troubleshooting 및 질의응답
- **세션 종료 후**:
  - Lab 결과 및 필요한 실습 이미지 정리
  - Quiz 답안 정리 (선택)
  - `members/<github-id>/weekNN/`에 작성 후 Pull Request 제출

---

## 📅 8주 차 커리큘럼

| 주차 | 주제 | 세부 내용 | 발표자 |
| :---: | :--- | :--- | :---: |
| **Week 1**<br>(8/31) | 사전 준비 | - 스터디 OT (자기소개, 스터디 규칙 및 GitHub 활용법)<br>- RHEL 10 / Rocky Linux 10 VM 환경 구축<br>- Shell 프롬프트 구조, 절대/상대 경로, 기본 명령어 (`ls`, `cd`, `mkdir`, `cp`, `mv`, `rm`) | 전체 |
| **Week 2**<br>(9/7) | 계정/그룹 관리 및<br>기본 파일 권한 | - 사용자 및 그룹 생성/수정/삭제 (`useradd`, `usermod`, `groupadd`)<br>- 비밀번호 만료 정책 설정 및 Sudo 권한 부여 (`/etc/sudoers.d/`)<br>- 표준 ugo/rwx 권한 설정 (`chmod`, `chown`) 및 `umask` 이해 | 김기석<br>이주희 |
| **Week 3**<br>(9/14) | 네트워크, Repos,<br>패키지 & SSH | - `nmcli`를 활용한 IPv4/IPv6 주소 및 Hostname 설정<br>- DNF/RPM 리포지토리 구성 및 패키지 관리 (`dnf`, `rpm`)<br>- Flatpak 리포지토리 구성 및 패키지 관리<br>- SSH 서비스 설정 및 키 기반 SSH 인증 구성 | 김보겸<br>김성윤 |
| **Week 4**<br>(9/21) | 특수 권한, ACL,<br>탐색 & 아카이브 | - 특수 권한 (Setuid, Setgid, Sticky Bit) 및 협업 디렉토리 구축<br>- POSIX ACL 세부 권한 제어 (`setfacl`, `getfacl`)<br>- `find`, `grep` 정규표현식 활용 및 `tar`/`gzip`/`bzip2` 아카이브/압축<br>- Hard Link / Symbolic Link 생성 및 차이 이해 | 박시윤<br>미정 |
| **Week 5**<br>(9/28) | 로컬 스토리지, LVM,<br>파일 시스템 & AutoFS | - GPT 파티션 생성/삭제 (`fdisk`, `gdisk`)<br>- LVM (PV, VG, LV) 생성/제거 및 무손실 LV 확장 (`lvextend -r`)<br>- VFAT, ext4, XFS 생성, Swap 추가, UUID 기반 `/etc/fstab` 영구 마운트<br>- NFS 공유 마운트 및 AutoFS 자동 마운트 구성 | 이서영<br>박정우 |
| **Week 6**<br>(10/5) | Systemd 서비스,<br>방화벽 & SELinux | - Systemd 서비스 제어 및 부팅 시 자동 시작 설정 (`systemctl`)<br>- `firewall-cmd`를 이용한 영구(`--permanent`) 방화벽 규칙 적용<br>- SELinux 모드 전환, 파일 컨텍스트 (`semanage fcontext`, `restorecon`)<br>- SELinux 포트 레이블 관리 및 Booleans 설정 변경 | 미정 |
| **Week 7**<br>(10/12) | 작업 예약, 로그 관리,<br>시간 동기화 & Tuned | - `at`, `crontab` 작업 예약 및 Systemd Timer 유닛 활용<br>- 프로세스 우선순위 조정 (`nice`, `renice`) 및 `tuned-adm` 프로파일 적용<br>- `journalctl` 저널 로그 분석 및 영구 보존 설정<br>- Chrony 타임 서비스(NTP) 클라이언트 설정 | 미정 |
| **Week 8**<br>(10/19) | 부팅 프로세스, Root 복구<br>& 종합 트러블슈팅 | - GRUB 부트로더 수정 및 기본 부팅 타겟 변경<br>- 부팅 중단 및 `rd.break` 진입을 통한 Root 패스워드 재설정<br>- 간단한 Shell Script 작성 (조건문 `if`, 루프문 `for`, 인자 `$1` 처리)<br>- 전체 실습 환경 재부팅 후 설정 유지 점검 및 종합 트러블슈팅 | 미정 |
