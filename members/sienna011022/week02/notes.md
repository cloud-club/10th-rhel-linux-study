계정/그룹 관리 및 기본 파일 권한

RH124 10장 로컬 사용자 및 그룹 관리
RH124 11장 파일에 대한 액세스 제어

실습 환경은 lima로 띄운 Rocky Linux 10 (M3 맥이라 aarch64)

### 시작점

계정 관리는 결국 파일 세 개를 고치는 일이다.

/etc/passwd : 사용자 목록. 누구나 읽을 수 있음
/etc/shadow : 비밀번호 해시와 만료 정책. 권한 000
/etc/group  : 그룹 목록

useradd, usermod, chage 전부 이 파일들을 대신 고쳐주는 도구다.
명령어를 외우는 것보다 파일 한 줄을 읽을 줄 아는 게 빠르다고 느꼈다.

vipw, vigr로 직접 편집할 수도 있는데 실무에서는 굳이.

### 사용자

/etc/passwd 엔트리

  이름:x:UID:GID:설명:홈디렉터리:셸

x는 예전에 여기 해시가 들어가던 자리. 지금은 shadow로 옮겨서 x만 남았다.
맥북 /etc/passwd 열어보니 x 자리에 *가 있었는데, 맥은 이 파일이 진짜 DB가 아니고
Open Directory를 쓰기 때문. dscl로 봐야 실제 계정이 보인다.

UID 범위

  0       root
  1-200   Red Hat이 미리 정해둔 시스템 계정. 어느 RHEL에서든 같은 번호
  201-999 패키지 설치할 때 999부터 아래로 내려가며 할당. 호스트마다 다름
  1000-   일반 사용자

201~999가 호스트마다 다르다는 게 포인트. NFS로 파일 공유할 때 양쪽 UID가 어긋나면
소유자가 엉뚱하게 보인다. ← 5주차에 확인해볼 것

VM에서 실제로 보니 sshd가 74(1-200 구간), chrony가 994(201-999 구간)였다.

useradd -m -s /bin/bash -G devteam devuser

  -m 홈 디렉터리 생성 + /etc/skel 복사
  -s 로그인 셸
  -G 보조 그룹

셸을 /sbin/nologin으로 주면 로그인은 막고 서비스 계정으로만 쓴다.
맥은 같은 용도로 /usr/bin/false를 쓴다.

usermod

  -aG  보조 그룹 추가       ← a 빼면 기존 그룹 다 날아감
  -L / -U  잠금 / 해제
  -l   이름 변경

실무에서는 퇴사자 계정을 지우지 않고 usermod -L로 잠근다고 한다.
지워버리면 그 사람 UID가 재사용되면서 남아있던 파일 소유권이 새 사람한테 넘어간다.

userdel과 userdel -r

  userdel     계정만 삭제. 홈 디렉터리는 남음
  userdel -r  홈 디렉터리까지

-r 없이 지우면 홈 디렉터리가 소유자 없이 남고 ls -l에 UID 숫자로 보인다.
이 상태에서 같은 UID로 새 사용자를 만들면 그 파일들을 그대로 소유하게 된다.

### 그룹

기본 그룹(primary)과 보조 그룹(supplementary)의 차이가 제일 헷갈렸다.

기본 그룹은 도장. 내가 새 파일을 만들면 그 그룹이 찍힌다.
보조 그룹은 열쇠. 이미 있는 파일에 접근할 때만 쓴다.

여기서 한 번 틀렸다. alice를 devteam 보조 그룹에 넣고 파일을 만들면
그룹이 devteam이 될 줄 알았는데 아니다. alice의 기본 그룹인 alice가 찍힌다.

RHEL은 useradd할 때 같은 이름의 그룹을 자동으로 만든다. UPG(User Private Group).
그래서 기본 그룹을 따로 지정 안 하면 alice의 기본 그룹은 alice다.

  groupadd -g 5000 devteam   GID 지정
  groupadd -r                시스템 그룹 (999 이하)
  groupmod -n 새이름 옛이름
  groupdel                   누군가의 기본 그룹이면 거부됨

여러 명이 같이 쓰는 디렉터리에서 그룹이 계속 유지되게 하려면 setgid를 건다.
chmod g+s. ← 4주차

### 비밀번호

/etc/shadow 필드

  이름:해시:마지막변경일:최소:최대:경고:비활성:만료:예약

chage 옵션이 4~6번 필드와 그대로 대응된다.

  chage -m 1 -M 60 -W 7 devuser
  chage -l devuser

해시 형식

  $y$j9T$소금$해시

  $y  yescrypt (RHEL 9 이후 기본)
  $6  SHA-512
  $5  SHA-256

가운데 소금(salt)은 무작위 값. 같은 비밀번호를 써도 사용자마다 해시가 달라진다.
미리 계산해둔 해시 표로 뚫는 공격을 막기 위한 것.

passwd -l 은 해시 앞에 !를 붙인다. 해시는 그대로 두고 매칭만 실패하게 만드는 방식이라
passwd -u로 원래대로 돌릴 수 있다.

useradd로 만들고 비밀번호를 아직 안 준 계정은 shadow에 !만 들어있다.
그래서 passwd -S가 L(Locked)로 나온다. NP는 필드가 아예 빈 경우.

### root와 sudo

su   대상 계정의 비밀번호를 안다
sudo 내 비밀번호를 쓰고, 규칙에 적힌 만큼만 빌린다

su -  는 대상 사용자의 환경까지 새로 읽는다. -가 있고 없고가 다르다.
UID 0은 권한 비트 검사 자체를 건너뛴다. 그래서 root는 000인 /etc/shadow도 읽는다.

sudoers 한 줄 읽는 법

  %adminteam ALL=(ALL:ALL) ALL
  ---------- --- --- --- ---
  누가        어느 어떤  어떤 어떤
             호스트 사용자 그룹 명령

그룹 앞에 %. 안 붙이면 그 이름의 사용자를 찾는다.

/etc/sudoers를 직접 고치지 않고 /etc/sudoers.d/ 아래에 파일을 따로 두는 이유는
패키지 업데이트로 원본이 덮여도 규칙이 남기 때문. 권한은 0440.
문법 틀리면 sudo가 통째로 안 되므로 visudo -cf로 먼저 검사한다.

sudo whoami가 root를 찍고, 그 다음 whoami는 다시 원래 사용자다.
sudo는 그 명령 하나에만 붙는다. 셸 자체를 root로 바꾸려면 sudo -i.

### 권한

숫자 모드는 통째로 덮어쓰고, 기호 모드는 해당 부분만 더하거나 뺀다.
기존 설정을 건드리면 안 될 때는 기호 모드를 써야 한다.

  chmod 644 report.txt
  chmod g+w report.txt
  chown kim:dev project/

기본값이 파일 666, 디렉터리 777인 이유는 디렉터리에는 x가 있어야 들어갈 수 있기 때문.
umask는 여기서 뺀다. 022면 파일 644, 디렉터리 755.

디렉터리의 x는 실행이 아니라 진입으로 읽어야 한다.

sticky bit

  chmod 1777 /shared/dropbox

파일 삭제는 파일 권한이 아니라 그 파일이 든 디렉터리의 쓰기 권한으로 결정된다.
그래서 공용 디렉터리에서는 남의 파일도 지워진다. sticky bit를 붙이면
파일 소유자와 root만 지울 수 있다. ls -ld로 보면 맨 끝이 t.

/tmp가 실제로 drwxrwxrwt다.

### 실습하다 막힌 것

useradd: Can't get unique subordinate UID range

lima가 컨테이너용으로 내 계정에 서브 UID를 10억 개 넘게 할당해놔서
/etc/login.defs의 SUB_UID_MAX(600100000)를 넘어버린 상태였다.

  /etc/subuid  sienna:524288:1073741824
  SUB_UID_MAX  600100000

처음엔 상한을 올릴까 했는데, 안 쓰는 범위를 줄이는 게 맞다.
표준 할당량인 65536으로 고쳤더니 해결됐다.

  sienna:524288:65536
  alice:589824:65536
  bob:655360:65536

서브 UID는 rootless 컨테이너에서 컨테이너 안 root(0번)를 호스트의 일반 UID로
매핑할 때 쓰는 범위다. ← 17장 Podman에서 다시 나올 듯

### 확인해볼 것

- setgid 걸린 공유 디렉터리 직접 만들어보기 (4주차)
- ls -l /etc/shadow 가 000인데 일반 사용자가 passwd로 어떻게 자기 비밀번호를 바꾸는지
  ls -l $(which passwd) 해보면 s가 보인다 ← setuid
- 201-999 UID가 호스트마다 다른 게 NFS에서 실제로 어떻게 깨지는지 (5주차)
