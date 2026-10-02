# 실습

LVM

## 문제

### 1. 다음 중 용량을 늘리기 위해 기존 VG에 PV를 추가하는 명령은 무엇입니까?
- a. `vgexpand /dev/vg01 /dev/sdb3`
- b. `vgextend /dev/sdb3 /dev/vg01`
- c. `vgextend /dev/vg01 /dev/sdb3`
- d. `vggrow /dev/sdb3 /dev/vg01`

### 2. pvmove 명령은 어떤 작업을 수행합니까?
- a. `PV에서 모든 메타데이터를 삭제합니다.`
- b. `VG에서 PV를 제거합니다.`
- c. `시스템에서 PV를 삭제합니다.`
- d. `한 PV의 확장 영역에서 다른 PV의 확장 영역으로 데이터를 이동합니다.`

### 3. 다음 중 용량을 줄이기 위해 VG에서 PV를 제거하는 명령은 무엇입니까?
- a. `vgreduce /dev/vg01 /dev/sdb3`
- b. `vgremove /dev/vg01 /dev/sdb3`
- c. `vgreduce /dev/sdb3`
- d. `vgrecall /dev/vg01 /dev/sdb3`

### 4. 참 또는 거짓: lvremove, vgremove, pvremove 명령은 되돌릴 수 있는 작업입니다.
- a. `참`
- b. `LVM 구성요소에 따라 다름`
- c. `거짓`

### 5. 전체 LVM 구성 요소 구조를 제거하는 올바른 순서는 무엇입니까?
- a. `umount > lvremove > vgremove > pvremove`
- b. `lvremove > umount > pvremove > vgremove`
- c. `vgremove > pvremove > lvremove > umount`
- d. `pvremove > vgremove > lvremove > umount`
---

# 정답

## LVM(논리 볼륨 관리자) 실습 해설

### 1. 다음 중 용량을 늘리기 위해 기존 VG에 PV를 추가하는 명령은 무엇입니까?

**정답:** **c. `vgextend /dev/vg01 /dev/sdb3**`

* **핵심 포인트:** 기존 볼륨 그룹(VG)의 용량을 확장하기 위해 새로운 물리 볼륨(PV)을 추가하는 명령어와 올바른 인자 순서(`명령어 VG명 PV명`)를 묻는 문제입니다.
* **선지 설명:**
* **a. `vgexpand /dev/vg01 /dev/sdb3`:** `vgexpand`라는 명령어는 존재하지 않습니다.
* **b. `vgextend /dev/sdb3 /dev/vg01`:** 명령어는 맞으나 인자 순서가 잘못되었습니다. 볼륨 그룹명이 먼저 와야 합니다.
* **d. `vggrow /dev/sdb3 /dev/vg01`:** `vggrow`라는 명령어는 존재하지 않습니다.

---

### 2. pvmove 명령은 어떤 작업을 수행합니까?

**정답:** **d. 한 PV의 확장 영역에서 다른 PV의 확장 영역으로 데이터를 이동합니다.**

* **핵심 포인트:** `pvmove` 명령어의 목적을 이해하는 문제입니다. 노후화된 디스크 교체나 제거를 위해 특정 물리 볼륨(PV)에 저장된 물리 확장 영역(PE, 데이터)을 동일한 VG 내의 다른 빈 PV로 안전하게 옮길 때 사용합니다.
* **선지 설명:**
* **a. PV에서 모든 메타데이터를 삭제합니다:** 이는 `pvremove` 명령어가 수행하는 작업입니다.
* **b. VG에서 PV를 제거합니다:** 이는 `vgreduce` 명령어가 수행하는 작업입니다.
* **c. 시스템에서 PV를 삭제합니다:** 물리적인 삭제나 메타데이터 초기화는 `pvremove`에 해당합니다.

---

### 3. 다음 중 용량을 줄이기 위해 VG에서 PV를 제거하는 명령은 무엇입니까?

**정답:** **a. `vgreduce /dev/vg01 /dev/sdb3**`

* **핵심 포인트:** 볼륨 그룹에서 특정 물리 볼륨을 빼내어 전체 용량을 축소하는 명령어(`vgreduce`)와 올바른 문법(`vgreduce VG명 PV명`)을 묻는 문제입니다.
* **선지 설명:**
* **b. `vgremove /dev/vg01 /dev/sdb3`:** `vgremove`는 볼륨 그룹 자체를 완전히 삭제하는 명령어입니다. 특정 PV만 빼내는 것이 아닙니다.
* **c. `vgreduce /dev/sdb3`:** 문법이 틀렸습니다. 대상이 되는 볼륨 그룹 이름이 누락되었습니다.
* **d. `vgrecall /dev/vg01 /dev/sdb3`:** `vgrecall`이라는 명령어는 존재하지 않습니다.

---

### 4. 참 또는 거짓: lvremove, vgremove, pvremove 명령은 되돌릴 수 있는 작업입니다.

**정답:** **c. 거짓**

* **핵심 포인트:** LVM 구성 요소를 삭제하는 `remove` 계열 명령어들의 파괴적이고 비가역적인 특성을 묻는 문제입니다. 실행 즉시 구조와 메타데이터가 파괴되므로 되돌릴 수 없습니다.
* **선지 설명:**
* **a. 참:** 삭제 작업은 기본적으로 복구가 불가능하므로 오답입니다.
* **b. LVM 구성 요소의 유형에 따라 다릅니다:** 구성 요소의 종류와 무관하게 모든 LVM 삭제 명령어는 파괴적인 작업입니다.

---

### 5. 전체 LVM 구성 요소 구조를 제거하는 올바른 순서는 무엇입니까?

**정답:** **a. `umount` > `lvremove` > `vgremove` > `pvremove**`

* **핵심 포인트:** LVM 구조를 안전하게 해체하려면 구성 요소를 생성했던 순서의 정확한 **역순**으로 진행해야 한다는 원칙을 묻는 문제입니다.
* **선지 설명:**
* **b. `lvremove` > `umount` > `pvremove` > `vgremove`:** 파일 시스템이 마운트된 상태에서는 논리 볼륨(LV)을 삭제할 수 없으며, VG보다 PV를 먼저 삭제할 수도 없습니다.
* **c. `vgremove` > `pvremove` > `lvremove` > `umount`:** 상위 논리 구조인 LV가 존재하는 상태에서 하위 구조인 VG를 먼저 삭제할 수 없습니다.
* **d. `pvremove` > `vgremove` > `lvremove` > `umount`:** 생성 순서와 동일하게 가장 밑바탕(PV)부터 지우려고 시도하는 잘못된 접근입니다.

---

## 요약

| 문제 | 핵심 |
| --- | --- |
| 1 | VG 확장 명령어(`vgextend`) 및 인자 순서 |
| 2 | `pvmove` 명령어의 데이터 마이그레이션 역할 |
| 3 | VG 축소 명령어(`vgreduce`) |
| 4 | LVM 삭제 명령어의 비가역성(파괴성) |
| 5 | LVM 구조의 안전한 해체 순서 (생성의 역순) |