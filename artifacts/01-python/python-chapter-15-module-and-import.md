# Chapter 15: module과 import

## 이 장에서 배우는 것

- module이 무엇인지, 왜 코드를 파일로 나누는지
- `import`로 다른 파일의 코드를 가져오는 방법
- `from ... import ...` 문법 사용하기
- Python 표준 라이브러리 활용하기
- `if __name__ == "__main__"` 패턴 이해하기

---

## 먼저 쉬운 설명

코드가 길어지면 한 파일에 모두 담기 어려워진다.

책도 한 권에 모든 내용을 다 담지 않고 챕터로 나누듯이, 코드도 역할별로 파일을 나눠서 관리한다.

**module = Python 코드가 담긴 `.py` 파일**

`greetings.py` 라는 module에 인사 관련 함수를 담아두면, 다른 파일에서 그 함수를 가져다 쓸 수 있다.

---

## 1. 직접 만든 module 사용하기

### module 만들기

`greetings.py` 파일을 만든다:

```python
# greetings.py

def say_hello(name):
    return f"안녕하세요, {name}님!"

def say_goodbye(name):
    return f"안녕히 가세요, {name}님!"
```

### module 가져오기

같은 폴더에 `main.py` 파일을 만든다:

```python
# main.py

import greetings

print(greetings.say_hello("Mina"))
print(greetings.say_goodbye("Tom"))
```

출력:
```
안녕하세요, Mina님!
안녕히 가세요, Tom님!
```

`import 파일이름` 으로 가져오고, `파일이름.함수이름()` 으로 사용한다.

---

## 2. from ... import ... 문법

필요한 것만 골라서 가져올 수 있다.

```python
from greetings import say_hello

print(say_hello("Mina"))   # greetings. 없이 바로 사용
```

여러 개 가져오기:

```python
from greetings import say_hello, say_goodbye
```

전부 가져오기 (권장하지 않음):

```python
from greetings import *   # 이름 충돌 가능성 있음
```

---

## 3. 별칭(alias) 사용하기

module 이름이 길면 별칭을 붙여서 짧게 쓸 수 있다.

```python
import greetings as g

print(g.say_hello("Mina"))
```

```python
from greetings import say_hello as hello

print(hello("Mina"))
```

---

## 4. Python 표준 라이브러리

Python에는 기본으로 포함된 module들이 있다. 설치 없이 바로 쓸 수 있다.

### 자주 쓰는 표준 라이브러리

```python
import math
print(math.sqrt(16))    # 4.0
print(math.pi)          # 3.141592...

import random
print(random.randint(1, 10))   # 1~10 사이 랜덤 숫자
print(random.choice(["a", "b", "c"]))

import datetime
today = datetime.date.today()
print(today)   # 2026-05-15

import os
print(os.getcwd())   # 현재 작업 디렉토리
```

---

## 5. `if __name__ == "__main__"` 패턴

```python
# calculator.py

def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

if __name__ == "__main__":
    print(add(3, 5))       # 직접 실행할 때만 실행됨
    print(subtract(10, 4))
```

`if __name__ == "__main__":` 블록은 이 파일을 **직접 실행할 때만** 실행된다.

다른 파일에서 `import calculator` 로 가져올 때는 이 블록이 실행되지 않는다.

**왜 이 패턴이 중요한가:**

```python
# main.py 에서 calculator를 가져올 때
import calculator   # add, subtract 함수만 가져옴
                    # if __name__ == "__main__" 블록은 실행 안 됨

result = calculator.add(10, 20)
```

---

## 6. 폴더로 module 구성하기 (패키지)

```
my_project/
├── main.py
├── utils/
│   ├── __init__.py    ← 패키지임을 알리는 빈 파일
│   ├── greetings.py
│   └── calculator.py
```

`utils` 폴더의 module 가져오기:

```python
from utils.greetings import say_hello
from utils import calculator
```

---

## 7. 따라 하기 실습

### 실습 1. module 분리 실습

아래 구조로 파일을 만든다:

```
memo_app/
├── main.py
├── memo_service.py
└── file_handler.py
```

`memo_service.py`:
```python
memos = []

def add(text):
    memos.append(text)

def show():
    for i, m in enumerate(memos, 1):
        print(f"{i}. {m}")
```

`main.py`:
```python
import memo_service

memo_service.add("Python 공부")
memo_service.add("GitHub 실습")
memo_service.show()
```

### 실습 2. 표준 라이브러리 활용

`random_quiz.py` 파일을 만들고:

```python
import random

questions = [
    ("Python에서 출력 함수는?", "print"),
    ("리스트에서 마지막 요소 인덱스는?", "-1"),
    ("조건문 키워드는?", "if"),
]

q, a = random.choice(questions)
answer = input(f"Q: {q}\n답: ")

if answer.strip() == a:
    print("정답!")
else:
    print(f"오답. 정답은 '{a}'입니다.")
```

---

## 자주 하는 실수

| 실수 | 증상 | 해결 방법 |
|------|------|----------|
| 파일이 다른 폴더에 있음 | `ModuleNotFoundError` | 같은 폴더에 두거나 경로 설정 |
| module 이름과 파일 이름 다름 | `ModuleNotFoundError` | `import 파일이름` (`.py` 제외) |
| `from module import *` 남용 | 이름 충돌, 어디서 왔는지 모름 | 필요한 것만 명시적으로 import |
| `__init__.py` 없음 | 패키지를 module로 인식 못 함 | 폴더 안에 빈 `__init__.py` 생성 |

---

## 확인 체크리스트

- [ ] `.py` 파일을 만들고 다른 파일에서 `import`할 수 있는가
- [ ] `import module` 과 `from module import func` 의 차이를 설명할 수 있는가
- [ ] `if __name__ == "__main__"` 이 왜 필요한지 설명할 수 있는가
- [ ] `random`, `math`, `datetime` 중 하나를 사용하는 코드를 작성할 수 있는가

---

## 한 번 더 생각해 보기

1. 모든 코드를 `main.py` 하나에 넣으면 어떤 문제가 생길까?
2. `import *` 을 피해야 하는 이유는 무엇인가?
3. `__init__.py` 파일이 없으면 어떻게 될까?

---

## 다음 장

Python 기초 챕터를 모두 마쳤다. 다음은 GitHub로 넘어가서 작성한 코드를 저장하고 협업하는 방법을 배운다.

---

## 참고 자료

- Python Tutorial: Modules — https://docs.python.org/3/tutorial/modules.html
- Python Standard Library — https://docs.python.org/3/library/index.html
