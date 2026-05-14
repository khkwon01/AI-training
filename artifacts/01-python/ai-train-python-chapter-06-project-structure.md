# ai-train Python Basics Chapter 04

## 이 장에서 배우는 것

- Python 파일과 폴더를 왜 나누는지
- `main.py`, `utils.py`, `data/`, `tests/`가 어떤 역할을 하는지
- 이름을 쉽게 짓는 기본 규칙
- `snake_case`가 무엇인지
- `import`가 폴더 구조와 어떤 관계가 있는지

## 먼저 쉬운 설명

코드가 길어지면 한 파일에 모든 내용을 넣기가 어려워진다.

그래서 Python에서는:

- 역할이 다른 코드를 파일로 나누고
- 비슷한 파일은 폴더로 묶고
- 이름을 규칙 있게 정리해서

나중에 다시 봐도 이해하기 쉽게 만든다.

이걸 너무 어렵게 생각할 필요는 없다.  
초보자에게는 "책상 위를 정리하는 방법"처럼 생각하면 된다.

## 1. 자주 보는 파일과 폴더

### `main.py`

프로그램을 시작하는 파일로 자주 사용한다.

### `utils.py`

여러 곳에서 함께 쓰는 작은 함수들을 모아둘 때 자주 쓴다.

### `data/`

JSON, CSV, 메모 파일처럼 데이터를 모아두는 폴더로 생각하면 쉽다.

### `tests/`

코드가 제대로 동작하는지 확인하는 테스트를 모아두는 폴더다.

## 2. 이름 짓기 규칙

Python에서는 보통 변수와 함수 이름을 `snake_case`로 쓴다.

예:

```python
student_name = "Mina"

def calculate_score():
    return 100
```

`snake_case`는 단어 사이를 `_`로 잇는 방식이다.

### 왜 이렇게 쓰나?

- 읽기 쉽다
- 여러 사람이 함께 봐도 통일감이 있다
- Python에서 많이 쓰는 방식이다

## 3. class 이름은 어떻게 짓나

class는 보통 단어 첫 글자를 크게 쓰는 방식으로 적는다.

예:

```python
class StudentProfile:
    pass
```

## 4. import와 파일 구조

예를 들어 `utils.py`에 함수가 있다고 하자.

```python
def say_hello(name):
    return f"안녕하세요, {name}"
```

`main.py`에서 이렇게 가져와 쓸 수 있다.

```python
from utils import say_hello

print(say_hello("Mina"))
```

즉, 파일을 나누더라도 `import`를 사용하면 필요한 기능을 다시 가져와 쓸 수 있다.

### 짧은 이해 점검

1. `main.py`와 `utils.py`는 보통 같은 역할일까, 다른 역할일까?
2. 함수 이름으로 `calculate_score`와 `CalculateScore` 중 무엇이 더 자연스러울까?
3. 다른 파일에 있는 함수를 쓰고 싶을 때 보통 무엇을 할까?

## 5. 따라 하기 실습

### 실습 1. 파일 역할 나누기

아래처럼 구성해 보자.

- `main.py`
- `utils.py`
- `data/`
- `tests/`

### 실습 2. utils 함수 만들기

```python
def add(a, b):
    return a + b
```

### 실습 3. main.py에서 가져오기

```python
from utils import add

print(add(2, 3))
```

## 자주 하는 실수

- 파일 이름을 너무 어렵게 짓기
- 변수 이름 규칙이 매번 바뀌기
- import 경로를 헷갈리기
- 역할이 다른 코드를 한 파일에 모두 넣기

## 확인 체크리스트

- `main.py`와 `utils.py` 역할 차이를 말할 수 있는가
- `snake_case`가 무엇인지 설명할 수 있는가
- 간단한 `import`를 사용할 수 있는가
- 왜 폴더를 나눠야 하는지 설명할 수 있는가

## 한 번 더 생각해 보기

1. 파일을 나누면 어떤 점이 편해질까?
2. 이름 규칙이 없으면 어떤 문제가 생길까?
3. `import`가 왜 필요한가?

## 교사용 메모

- 강조: 이 장은 문법보다 "파일 역할 나누기"를 이해시키는 것이 먼저다.
- 막힘: `main.py`와 `utils.py` 역할 차이, `snake_case` 규칙, import 대상 이해에서 자주 멈춘다.
- 질문 1: `main.py`와 `utils.py`는 무엇이 다를까?
- 질문 2: `utils.py`의 함수를 `main.py`에서 쓰려면 무엇이 필요할까?
