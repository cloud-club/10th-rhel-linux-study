# RHCSA Study 제출 템플릿

매주 스터디 종료 후 개인 실습 결과를 제출할 때 사용하는 템플릿입니다.

## Lab 템플릿

문제 유형에 따라 아래 템플릿 중 하나를 선택해서 사용합니다.

### `lab-command-template.md`

개념 확인 또는 간단한 명령어 작성 문제에 사용합니다.

예:
- 권한 계산
- 명령어 작성
- 짧은 이론 답안
- 별도 실습 이미지가 필요하지 않은 문제

### `lab-practice-template.md`

실제 Linux 환경에서 직접 수행하는 실습 문제에 사용합니다.

예:
- 명령어 실행
- 터미널 결과 확인
- 실습 이미지 캡처
- Troubleshooting 기록

## Quiz 템플릿

### `quiz-template.md`

현장 Quiz 참여 내용을 정리할 때 사용합니다.

- 답안 작성
- 오답 및 개념 정리
- 배운 점 정리

Quiz 제출은 선택 사항입니다.

## 사용 방법

필요한 템플릿을 자신의 주차 폴더로 복사한 뒤 파일명을 변경하여 작성합니다.

예:

```bash
cp templates/lab-command-template.md members/<github-id>/weekNN/lab01.md
cp templates/lab-practice-template.md members/<github-id>/weekNN/lab02.md
cp templates/quiz-template.md members/<github-id>/weekNN/quiz.md

```


## 제출 구조

```text
members/<github-id>/weekNN/
├── lab01.md
├── lab02.md
├── quiz.md        # 선택
└── images/        # 필요한 경우
