# 3주차 내용이 실무에서 나타나는 곳

개념이 비슷하다는 유추가 아니라, **같은 파일과 같은 설정이 그대로 쓰이는** 두 가지입니다.

---

## 1. `/etc/resolv.conf` 는 쿠버네티스 DNS의 뿌리입니다

18장에서 본 이름 확인 구조가 클러스터에서 두 층으로 반복됩니다.

### 파드 안쪽 — `search` 와 `ndots`

파드의 `/etc/resolv.conf`는 kubelet이 만들어 넣습니다.

```
nameserver 10.96.0.10          ← CoreDNS 서비스 IP
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

`search` 줄이 있어서 `curl my-service` 같은 짧은 이름이 동작합니다.
18장의 그 `search`입니다. 서비스 FQDN도 같은 구조로 쪼개집니다.

```
my-service.default.svc.cluster.local.
────────── ─────── ─── ───────────── ─
  서비스명  네임스페이스 타입  클러스터 도메인  루트
```

`ndots:5`가 성능 함정입니다. 점이 5개 미만인 이름은 `search` 도메인을 전부 붙여서 먼저 시도합니다.

```
curl https://api.github.com  을 호출하면

api.github.com.default.svc.cluster.local   NXDOMAIN
api.github.com.svc.cluster.local           NXDOMAIN
api.github.com.cluster.local               NXDOMAIN
api.github.com.ec2.internal                NXDOMAIN
api.github.com                             ← 5번째에야 성공
```

외부 API를 많이 호출하는 서비스면 DNS 쿼리가 5배로 나갑니다.
CoreDNS 부하와 응답 지연의 흔한 원인입니다.

해결은 이름 끝에 점을 찍거나(`api.github.com.`), 파드의 `ndots`를 낮추는 것입니다.

```yaml
spec:
  dnsConfig:
    options:
    - name: ndots
      value: "2"
```

18장에서 "끝의 점이 루트 도메인이고, 점이 있으면 `search`를 붙이지 않는다"고 한 것이
여기서 성능 문제로 직결됩니다.

### 노드 바깥쪽 — CoreDNS의 업스트림

CoreDNS 기본 설정(Corefile)에 이 줄이 있습니다.

```
forward . /etc/resolv.conf
```

클러스터 내부 이름이 아닌 건 전부 노드의 `/etc/resolv.conf`에 적힌 nameserver로 넘깁니다.
18장에서 NetworkManager가 관리한다고 한 그 파일이 클러스터 DNS의 업스트림이 됩니다.

그래서 노드의 resolv.conf가 잘못되면 이렇게 나타납니다.

```
서비스끼리는 잘 통신됨            ← 클러스터 내부라 CoreDNS가 직접 답함
외부 도메인 조회만 실패           ← forward 대상이 잘못됨
이미지 풀 실패, 외부 API 호출 실패
```

증상이 "일부만 안 되는" 형태라 원인 찾기가 어렵습니다.

유명한 변형이 하나 더 있습니다. systemd-resolved를 쓰는 노드(Ubuntu 등)는
`/etc/resolv.conf`가 `127.0.0.53` 스텁을 가리킵니다.
CoreDNS가 거기로 forward하면 자기 자신에게 되묻는 루프가 생기고,
CoreDNS의 `loop` 플러그인이 이를 감지해서 파드가 CrashLoopBackOff에 빠집니다.

```
[FATAL] plugin/loop: Loop (127.0.0.1:xxxx -> :53) detected for zone "."
```

RHEL/Rocky는 NetworkManager가 실제 nameserver를 `/etc/resolv.conf`에 써주기 때문에
이 문제가 덜합니다. RHEL 계열을 노드로 쓰는 실질적 이점 중 하나입니다.

### 정리

```
18장:  /etc/resolv.conf 의 search 와 nameserver
         ↓                    ↓
파드:  search → ndots:5    노드: nameserver → CoreDNS forward 대상
       쿼리 증폭                   외부 조회 전체가 여기 달림
```

디버깅할 때 파드 안에서 쓰는 명령도 18장 그대로입니다.

```bash
kubectl exec -it POD -- cat /etc/resolv.conf
kubectl exec -it POD -- getent hosts my-service
```

`getent`를 쓰는 이유도 같습니다. `dig`는 `/etc/hosts`와 `nsswitch` 순서를 무시하기 때문에,
애플리케이션이 실제로 어떻게 해석하는지 보려면 `getent`여야 합니다.

---

## 2. 리포지토리 설정이 CI를 깨뜨립니다

12장의 `.repo` 파일 설정이 실무에서 직접 사고로 이어진 사례입니다.

### 증상

Amazon Linux 2023에서 CI 빌드 중 Grafana Alloy를 설치하는데,
**10번에 1번꼴로 간헐적으로** 실패했습니다.

```
Failed to download metadata for repo 'grafana':
repomd.xml GPG signature verification error: Bad GPG signature
```

repo 설정은 이랬습니다.

```ini
[grafana]
name=grafana
baseurl=https://rpm.grafana.com
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://rpm.grafana.com/gpg.key
```

### `gpgcheck` 와 `repo_gpgcheck` 는 다릅니다

검사 대상이 둘로 나뉩니다.

```
gpgcheck=1        내려받은 .rpm 패키지 각각의 서명을 검증
repo_gpgcheck=1   리포지토리 메타데이터(repomd.xml)의 서명을 검증
```

에러 메시지가 `repomd.xml GPG signature verification error`이므로
패키지가 아니라 메타데이터 단계에서 터진 것입니다. 아직 패키지는 받지도 않았습니다.

`repo_gpgcheck`는 dnf 기본값이 `0`입니다. Grafana가 보안을 위해 켜두라고 문서에 안내하고 있어서
이 설정이 들어가 있었습니다.

메타데이터 검증은 파일 두 개를 대조하는 방식입니다.

```
repomd.xml       패키지 목록의 색인
repomd.xml.asc   그 색인에 대한 분리 서명(detached signature)
```

### 왜 간헐적인가

항상 실패하면 키가 틀린 것입니다. **10번에 1번이면 키 문제가 아닙니다.**

`repomd.xml`과 `repomd.xml.asc`를 따로 두 번 요청하는데, 그 사이에
CDN 엣지 노드가 달라지거나 메타데이터가 갱신되면 짝이 안 맞는 조합을 받게 됩니다.
서명은 옛 색인 것인데 색인은 새것이면 검증이 실패합니다.

```
요청 1: repomd.xml       → 엣지 A (새 메타데이터)
요청 2: repomd.xml.asc   → 엣지 B (옛 서명)
        → Bad GPG signature
```

로컬 dnf 캐시에 옛 `repomd.xml`이 남아 있는데 서명만 새로 받아도 같은 일이 생깁니다.

커뮤니티 포럼에도 같은 증상이 여러 건 올라와 있고 공식 해결책은 없습니다.
업스트림 CDN 사정이라 클라이언트가 고칠 수 있는 문제가 아니기 때문입니다.

### 대응

```bash
# 1. 캐시 정리 후 재시도 (완화책)
dnf clean all && dnf makecache
```

```bash
# 2. CI에 재시도 로직
for i in 1 2 3; do dnf install -y alloy && break || sleep 5; done
```

```ini
# 3. 메타데이터 검증만 끄기 (패키지 서명 검증은 유지)
repo_gpgcheck=0
gpgcheck=1
```

```
# 4. 사내 미러에 스냅샷 (근본 해결)
```

3번은 절충안입니다. 메타데이터 무결성은 포기하지만 패키지 서명 검증은 남으므로
위조된 패키지가 설치되지는 않습니다.

### 사내 미러를 두는 진짜 이유

보통 두 가지를 말합니다.

```
1. 속도    가까운 데서 받는다
2. 폐쇄망   외부로 못 나간다
```

그런데 실무에서 가장 자주 겪는 건 세 번째입니다.

```
3. 업스트림 사정으로 빌드가 깨지지 않게 한다
```

이번 사례가 정확히 그것입니다. 코드도 설정도 안 바뀌었는데 CI가 10%씩 실패했습니다.
미러에 스냅샷을 떠두면 업스트림 CDN이 뭘 하든 빌드는 그대로 돕니다.

Nexus나 Harbor 같은 도구는 yum 리포지토리와 컨테이너 이미지를 같은 서버에서 서빙합니다.
비유가 아니라 같은 물건입니다.

### 재현 가능성

같은 문제의 다른 얼굴입니다.

```
dnf install nginx                  언제 빌드하느냐에 따라 버전이 다르다
dnf install nginx-1.26.2-3.el10    고정

FROM nginx:latest                  태그는 덮어써질 수 있다
FROM nginx@sha256:abc...           다이제스트로 고정
```

12장에서 패키지 이름이 `name-version-release.arch` 4요소로 쪼개져 있는 이유가 여기 있습니다.
`release`까지 지정해야 빌드가 재현됩니다.
