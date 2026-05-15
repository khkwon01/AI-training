# ai-train Python 학습 지도

## 목표

VS Code 환경 준비부터 Python 기초 문법까지, 초보자가 어떤 순서로 공부하면 좋은지 안내한다.

---

## 권장 학습 순서

### 0단계. 환경 준비

Python을 시작하기 전에 먼저 개발 환경을 갖춘다.

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 1 | [VS Code 설치와 기본 사용](../01-python/ai-train-python-vscode-chapter-01.md) | VS Code 설치, 폴더 열기, 터미널, Python 확장 설치 |
| 2 | [Python 설치와 VS Code 설정](../01-python/ai-train-python-vscode-chapter-02-python-setup.md) | Python 설치, 인터프리터 선택, venv 생성 및 연결 |

---

### 1단계. 기초 문법 익히기

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 3 | [Chapter 01: print와 실행](../01-python/ai-train-python-chapter-01-print-and-run.md) | `print()`, 파일 실행, 따옴표 |
| 4 | [Chapter 02: 변수](../01-python/ai-train-python-chapter-02-variables.md) | 변수 만들기, 값 저장, 이름 규칙 |
| 5 | [Chapter 03: 자료형](../01-python/ai-train-python-chapter-03-data-types.md) | int, float, str, bool, f-string, type() |
| 6 | [Chapter 11: 조건문](../01-python/ai-train-python-chapter-11-conditions.md) | if/elif/else, 비교 연산자, 논리 연산자 |
| 7 | [Chapter 12: 반복문](../01-python/ai-train-python-chapter-12-loops.md) | for, while, range, break/continue, enumerate |
| 8 | [Chapter 13: 함수](../01-python/ai-train-python-chapter-13-functions.md) | def, 매개변수, return, 기본값 |
| 9 | [Chapter 14: class와 object](../01-python/ai-train-python-chapter-14-class-and-object.md) | class, __init__, self, 메서드 |
| 10 | [Chapter 15: module과 import](../01-python/ai-train-python-chapter-15-module-and-import.md) | import, from, 표준 라이브러리, __name__ |

---

### 2단계. 파일과 데이터 다루기

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 6 | [Chapter 04: 파일과 JSON](../01-python/ai-train-python-chapter-04-files-and-json.md) | 파일 읽기/쓰기, JSON 저장·불러오기 |

---

### 3단계. 오류와 디버깅 익히기

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 7 | [Chapter 05: 디버깅](../01-python/ai-train-python-chapter-05-debugging.md) | 오류 메시지 읽기, NameError, TypeError, print 디버깅 |

---

### 4단계. 프로젝트 구조 이해하기

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 8 | [Chapter 06: 프로젝트 구조](../01-python/ai-train-python-chapter-06-project-structure.md) | `main.py`, `utils.py`, snake_case, import |

---

### 5단계. 실행 환경 이해하기

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 9 | [Chapter 07: 실행 환경](../01-python/ai-train-python-chapter-07-environment.md) | 환경변수, dependency, path |
| 10 | [Chapter 09: venv와 pip](../01-python/ai-train-python-chapter-09-venv-and-pip.md) | venv 만들기, pip, requirements.txt |

---

### 6단계. 예외 처리와 후속 문법

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 11 | [Chapter 08: 예외 처리](../01-python/ai-train-python-chapter-08-exceptions.md) | try/except, 예외 종류 |
| 12 | [Chapter 10: comprehension과 datetime](../01-python/ai-train-python-chapter-10-comprehension-datetime.md) | 리스트 컴프리헨션, 날짜/시간 다루기 |

---

### 7단계. 확인 문제

| 파일 | 내용 |
|------|------|
| [퀵 체크 1](../01-python/ai-train-python-quick-check-01.md) | Chapter 01~05 핵심 확인 문제 |
| [퀵 체크 2](../01-python/ai-train-python-quick-check-02.md) | Chapter 06~10 핵심 확인 문제 |
| [퀵 체크 정답](../01-python/ai-train-python-quick-check-answers.md) | 정답 및 해설 |

---

### 8단계. 종합 연습

| 파일 | 내용 |
|------|------|
| [venv/pip 실습](../01-python/ai-train-python-practice-venv-pip-01.md) | 실제 패키지 설치 실습 |
| [누적 실습 01](../01-python/ai-train-python-practice-cumulative-01.md) | Chapter 01~10 종합 실습 문제 |
| [누적 실습 정답](../01-python/ai-train-python-practice-cumulative-01-answers.md) | 정답 및 해설 |

---

## 학습 팁

- **0단계 (환경 준비)** 는 반드시 먼저 완료해야 한다. 환경이 안 되면 아무것도 실행할 수 없다.
- **1~3단계** 는 순서대로 진행하는 것이 좋다. 변수를 모르면 자료형이 어렵고, 자료형을 모르면 디버깅이 어렵다.
- **4~6단계** 는 "실제 프로젝트 감각"을 키우는 구간이다. 소규모 프로젝트를 만들면서 같이 익히면 더 잘 이해된다.
- **7~8단계** 는 앞 단계를 어느 정도 학습한 뒤에 진행한다.

## 자주 막히는 지점

| 챕터 | 막히는 부분 | 추천 대응 |
|------|-----------|----------|
| Chapter 01 | 터미널 위치와 파일 경로 | `pwd`, `ls` 명령으로 위치 확인 |
| Chapter 03 | 문자열 `"14"` vs 숫자 `14` | `type()` 으로 직접 확인 |
| Chapter 08 | `try/except` 언제 쓰는지 | 실제 오류를 일부러 만들어서 감각 익히기 |
| Chapter 09 | venv 활성화 여부 | 터미널 프롬프트 `(.venv)` 표시 확인 |

---

## 교사용 메모

- Chapter 01~03은 가장 많은 질문이 나오는 구간이므로 예시를 여러 개 준비하는 것이 좋다.
- Chapter 08~09는 실제 프로젝트에서 중요한 내용이지만 초보자에게 낯설어서 충분한 연습 시간이 필요하다.
- 필요한 경우 [Teacher Note Template](../01-python/ai-train-python-teacher-note-template.md)를 함께 사용한다.
