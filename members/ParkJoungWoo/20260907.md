사용자 및 그룹 생성, 수정, 삭제 (useradd, usermod, groupadd)

7.3 User Management Files

7.9 User Access Topics

비밀번호 만료 정책 설정 및 Sudo 권한 부여 (/etc/sudoers.d/)

2.6 Changing Your Password and Shell

2.19.2 /user 디렉토리

2.20 Sudo

표준 ugo/rwx 권한 설정 (chmod, chown) 및 umask 이해

2.17.1 File Modes and Permission

### 목표

자동차를 운전할 때 모터의 구동원리 보단 운전 방법이 중요하다.

금번 스터디에서도 리눅스의 동작 방식에 초점을 맞추고 내부 구동 원리를 찾는 것은 지양한다.

따라서 추상화를 통해 대략적인 동작 방식을 배우는 것을 목표로 한다.

리눅스에서 큰 추상은 아래와 같다.

User : GUI, Server, Shell

Kernel : System Call, Process Management, Memory Management, Device Driver

Hardware : Processor, Main Memory, Disk, Network

위 항목에 대해서도 자세하게 배울 예정이나 일단 Location의 개념으로 위 영역을 나누는 것으로 시작하자.

### 메인 개념

#### 1. 사용자 및 그룹 생성, 수정, 삭제 (useradd, usermod, groupadd)

- 사용자 생성
    - 기본 생성 : sudo useradd testuser
    - 홈 디렉토리 및 쉘 설정 : sudo useradd testuser -m -s /bin/bash -g 1000
    - G는 보조 그룹 : sudo useradd testuser5 -s /bin/bash -m  -g 1001 -G testuser
- 사용자 수정
    - 그룹 변경 : sudo usermod -g testgroup testuser
    - 사용자 이름 변경 : sudo usermod -l testuser0 testuser1
    - 홈 디렉토리 변경 : sudo usermod -d /home/testuser01 ← 디렉토리를 만들어야 함
    - 계정 잠금/해제 : sudo usermod -L testuser / sudo usermod -U testuser
- 사용자 삭제
    - 계정만 : sudo userdel testuser
    - 계정과 디렉토리 모두 : sudo userdel -r testuser
- 디폴트 값 조정
    - useradd -D : 현재 디폴트 값들 출력
    - useradd -D -s /bin/bash : bash 쉘로 디폴트 값 수정

./files/img.png

- 그룹 생성
    - 기본 생성 : sudo groupadd testgroup
    - guid 지정 : sudo groupadd testgroup -g 1001
- 그룹 수정
    - 그룹명 수정 : sudo groupmod -n newgroup oldgroup
    - 그룹 ID 수정 : sudo groupmod -g 7777 testgroup
- 그룹 삭제
    - sudo groupdel testgroup
    - 만약 특정 user의 primary group으로 지정된 group을 삭제한다면
        - sudo userdel -g testgroup
        - sudo groupdel testgroup

*사용자란?

리눅스에서는 ID로 사용자를 관리한다.

하지만 ID는 사람 친화적이지 않으므로 userspace(위에 기술한 영역)에서는 name을 활용한다.

/etc/passwd 파일은 단순히 ID ↔ name을 매핑한 것이다.

/etc/passwd 파일의 엔트리 구조는 아래와 같다.

login name:password:UserID:GroupID:GECOS(일반적인 유저 내용):home directory:shell

*그룹이란?

리눅스에서 제공하는 파일 공유 체계

하나의 컴퓨터를 다수의 유저가 공유할 때는 중요했으나 현재는 상대적으로 덜 중요해진 항목

/etc/group에 엔트리 정보가 있다. 참고로 엔트리 항목중에 비밀번호가 있는데 현재는 설정하지 않는 것이 일반적

groupname:password:GroupID:유저들

#### 2. 비밀번호 만료 정책 설정 및 Sudo 권한 부여 (/etc/sudoers.d/)

- 비밀번호란 만료 정책 설정 방법
    - 인터렉션으로 수정 : sudo chage testuser
    - 현재 설정 확인 : sudo chage -l testuser
    - 30일 마다 수정하도록 설정 : sudo chage -M 30 testuser
    - 비밀번호 변경 주기 설정(최소 대기 기간) : sudo chage -m 7 testuser
    - 비밀번호 만료 알림 주기 설정 : sudo chage -W 14 testuser
    - 비밀번호를 즉시 수정하도록 설정 : sudo chage -d 0 testuser
- sudo 권한 설정 방법
    - 일반적으로 visudo 명령으로 superuser를 관리
    - sudo visudo -f /etc/sudoers.d/testuser
    - 모든 명령어 권한 부여
        - testuser ALL=(ALL:ALL) ALL
    - systemctl, useradd에만 권한 부여
        - testuser ALL=(ALL) /usr/bin/systemctl, /usr/sbin/useradd

#### 3. 표준 ugo/rwx 권한 설정 (chmod, chown) 및 umask 이해

- chmod로 권한 설정
    - 파일의 기본 최대 권한 666
    - 디렉토리의 기본 최대 권한 777
        - PAM(Pluggable Authentication Modules)을 수정하면 된다 ← 다음 공부
    - 4 read + 2 write + 1 executive을 계산해 User/Group/Other 에게 적용
- chown으로 소유권 설정
    - 사용자의 소유권 수정 : sudo chown testuser testfile
    - 그룹의 소유권 수정 : sudo chgrp testgroup testfile
    - 소유자와 그룹을 수정 : sudo chown testuser:testgroup testfile
    - 재귀적 수정 : sudo chown -R testuser:testgroup testfile
- umask란?
    - 새로 생성되는 파일이나 디렉토리에 제거될 권한
    - 마이너스 권한이라고 생각하면 편함
