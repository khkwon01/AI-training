# ai-train Python Basics Chapter 05

## 이 장에서 배우는 것

- 환경변수가 무엇인지
- dependency가 무엇인지
- path를 왜 알아야 하는지
- Python 프로젝트를 실행할 때 왜 이 개념들이 필요한지

## 먼저 쉬운 설명

Python 프로그램은 코드만 있다고 바로 잘 실행되는 것이 아니다.

가끔은:

- 어떤 설정값이 필요한지
- 어떤 라이브러리를 설치해야 하는지
- 어느 파일을 읽어야 하는지

를 같이 알아야 한다.

이 장에서는 이 세 가지를 아주 쉬운 말로 설명한다.

## 1. 환경변수란 무엇인가

환경변수는 프로그램 바깥에서 주는 설정값이라고 생각하면 쉽다.

예:

- `APP_MODE=dev`
- `APP_MODE=prod`

Python에서는 이렇게 읽을 수 있다.

```python
import os

app_mode = os.environ.get("APP_MODE", "dev")
print(app_mode)
```

## 2. dependency란 무엇인가

dependency는 "내 프로그램이 동작하려면 함께 필요한 다른 라이브러리"를 뜻한다.

예를 들어:

- `json`은 Python 기본 기능이라 바로 쓸 수 있다
- 어떤 외부 라이브러리는 따로 설치해야 한다

## 3. 가상환경이란 무엇인가

가상환경은 프로젝트마다 필요한 라이브러리를 따로 관리하는 공간이다.

쉽게 말하면 프로젝트마다 다른 상자를 하나씩 만드는 것과 비슷하다.

## 4. path란 무엇인가

path는 파일이나 폴더의 위치를 말한다.

예:

- `data/student.json`
- `notes/today.txt`

Python에서는 `pathlib`를 쓰면 파일 위치를 좀 더 읽기 쉽게 다룰 수 있다.

```python
from pathlib import Path

file_path = Path("data") / "student.json"
print(file_path)
```

### 짧은 이해 점검

1. 코드를 바꾸지 않고 설정값만 바꾸고 싶을 때 무엇을 사용할 수 있을까?
2. 다른 컴퓨터에서 프로그램이 안 돌아가면 무엇이 빠졌는지 먼저 의심해 볼 수 있을까?
3. `Path("data") / "profile.json"`은 무엇을 만드는 코드일까?

## 5. 따라 하기 실습

### 실습 1. 환경변수 읽기

```python
import os

user_name = os.environ.get("USER_NAME", "guest")
print(user_name)
```

### 실습 2. pathlib로 경로 만들기

```python
from pathlib import Path

file_path = Path("data") / "profile.json"
print(file_path)
```

### 실습 3. 파일 존재 확인하기

```python
from pathlib import Path

file_path = Path("data") / "profile.json"
print(file_path.exists())
```

## 자주 하는 실수

- 환경변수가 항상 있다고 생각하기
- 필요한 라이브러리를 설치하지 않고 바로 실행하기
- path를 문자열로만 다루다가 오타를 내기
- 파일 위치가 맞는지 확인하지 않기

## 확인 체크리스트

- 환경변수가 무엇인지 설명할 수 있는가
- dependency가 왜 필요한지 설명할 수 있는가
- path가 무엇인지 말할 수 있는가
- `os.environ.get()`와 `Path()`를 간단히 사용할 수 있는가

## 한 번 더 생각해 보기

1. 코드를 바꾸지 않고 설정만 바꾸고 싶을 때 무엇을 쓰면 좋을까?
2. 내 컴퓨터에서는 되는데 다른 컴퓨터에서 안 되는 이유는 무엇일 수 있을까?
3. 파일 경로를 잘못 적으면 어떤 일이 생길까?

## 교사용 메모

- 강조: 환경변수, dependency, path를 각각 외우기보다 "실행에 필요한 설정, 준비물, 위치"로 묶어서 설명한다.
- 막힘: 환경변수와 Python 변수 차이, 기본 라이브러리와 외부 라이브러리 차이, 상대 경로 읽기에서 자주 막힌다.
- 질문 1: 환경변수는 코드 안의 값일까, 바깥에서 주는 값일까?
- 질문 2: `Path("data") / "profile.json"`은 어떤 위치를 뜻할까?

## 참고 자료

- Python `os` documentation
- Python `pathlib` documentation
- Python `venv` documentation
- Python Packaging User Guide
