# ai-train Python Basics Chapter 07

## 이 장에서 배우는 것

- `venv`가 무엇인지
- `pip`가 무엇인지
- 왜 프로젝트마다 환경을 나누는지
- 필요한 라이브러리를 어떻게 설치하는지

## 먼저 쉬운 설명

Python 프로그램을 만들다 보면 어떤 코드는 기본 Python만으로 되지만, 어떤 코드는 추가 라이브러리가 필요하다.

그래서 중요한 것은:

- 어떤 라이브러리가 필요한지 정리하고
- 프로젝트마다 다른 환경을 섞지 않는 것이다

이때 `venv`와 `pip`를 배운다.

## 1. `pip`란 무엇인가

`pip`는 Python 라이브러리를 설치하는 도구라고 생각하면 쉽다.

```bash
pip install requests
```

## 2. `venv`란 무엇인가

`venv`는 프로젝트마다 따로 쓰는 Python 상자라고 생각하면 쉽다.

이 상자를 쓰면 프로젝트마다 필요한 라이브러리를 따로 관리할 수 있다.

## 3. 왜 나누는 것이 중요할까

- 버전 충돌을 줄일 수 있다
- 프로젝트별 환경을 구분할 수 있다
- 다른 사람과 공유할 때 설명이 쉬워진다

## 4. 가상환경 만들기

```bash
python3 -m venv .venv
```

## 5. 가상환경 사용하기

macOS나 Linux에서는 보통 이렇게 실행한다.

```bash
source .venv/bin/activate
```

## 6. 라이브러리 설치하기

```bash
pip install requests
```

## 7. 설치 목록 정리하기

```bash
pip freeze > requirements.txt
```

이렇게 하면 현재 설치된 목록을 파일로 저장할 수 있다.

## 8. 따라 하기 실습

### 실습 1. 가상환경 만들기

```bash
python3 -m venv .venv
```

### 실습 2. 가상환경 활성화하기

```bash
source .venv/bin/activate
```

### 실습 3. 라이브러리 설치하기

```bash
pip install requests
```

### 실습 4. 설치 목록 저장하기

```bash
pip freeze > requirements.txt
```

## 교사용 메모

- 강조: `venv`는 환경을 나누는 도구, `pip`는 그 안에 설치하는 도구라고 짧게 구분한다.
- 막힘: 어떤 환경에 설치했는지 헷갈리거나 `requirements.txt`를 왜 남기는지 이해하지 못하는 경우가 많다.
- 질문 1: 왜 프로젝트마다 가상환경을 따로 만드는 것이 좋을까?
- 질문 2: `requirements.txt`는 왜 필요할까?

## 자주 하는 실수

- 가상환경을 만들지 않고 바로 설치하기
- 어떤 환경에 설치했는지 헷갈리기
- `requirements.txt`를 남기지 않기

## 확인 체크리스트

- `pip`가 무엇인지 설명할 수 있는가
- `venv`가 왜 필요한지 설명할 수 있는가
- 가상환경을 만들고 활성화할 수 있는가
- 설치 목록을 파일로 저장할 수 있는가

## 한 번 더 생각해 보기

1. 왜 프로젝트마다 환경을 나누면 좋을까?
2. 다른 사람과 프로젝트를 공유할 때 왜 설치 목록이 필요할까?
3. 가상환경을 쓰지 않으면 어떤 일이 생길 수 있을까?

---

## 더 알아보기: uv (선택 사항)

`uv`는 2024년부터 많이 쓰이는 빠른 Python 패키지 관리 도구다. `pip`와 `venv`를 따로 쓰는 대신 하나로 처리한다.

초보자는 `pip`와 `venv`를 먼저 익히고, 익숙해지면 `uv`를 써보는 것이 좋다.

```bash
# uv 설치 (Mac/Linux)
curl -LsSf https://astral.sh/uv/install.sh | sh

# 새 프로젝트 시작
uv init my_project
cd my_project

# 패키지 추가 (pip install 대신)
uv add requests

# 실행 (venv 활성화 없이도 가능)
uv run python main.py
```

| 항목 | pip + venv | uv |
|------|-----------|-----|
| 속도 | 보통 | 매우 빠름 |
| 명령어 수 | 여러 개 | 적음 |
| 초보자 권장 | 먼저 배우기 | 나중에 시도 |
