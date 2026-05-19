# Chapter 06. 프로젝트 구조와 모듈

## 이 장에서 배우는 것

- 파일과 폴더를 왜 나눠야 하는지
- 실제 메모 앱을 예시로 폴더 구조 설계하기
- `__init__.py`가 무엇이고 왜 필요한지
- 절대 import vs 상대 import의 차이
- 파일 이름 짓는 규칙
- `if __name__ == "__main__"` 패턴의 의미와 필요성
- `ImportError`, `ModuleNotFoundError` 실제 에러 대처법

---

## 왜 프로젝트 구조가 필요한가

처음에는 파일 하나에 모든 코드를 넣어도 괜찮다.  
하지만 코드가 100줄, 200줄을 넘어가면 문제가 생기기 시작한다.

- 어디에 무슨 코드가 있는지 찾기 어려워진다
- 같은 기능이 여러 곳에 중복된다
- 한 부분을 고치면 다른 부분이 망가진다
- 다른 사람이 코드를 읽기 어렵다

이 문제를 해결하는 방법이 **"역할에 따라 파일과 폴더를 나누는 것"**이다.  
책을 챕터와 섹션으로 나누는 것과 같은 원리다.

---

## 1. 파일 이름 짓는 규칙

Python에서 파일과 폴더 이름은 `snake_case`를 사용한다.

| 규칙 | 예시 |
|------|------|
| 소문자만 사용 | `memo_app.py` (O), `MemoApp.py` (X) |
| 단어 사이는 `_` | `save_memo.py` (O), `savememo.py` (X) |
| 의미 있는 이름 | `utils.py`, `storage.py` (O), `a.py`, `test1.py` (X) |
| 숫자로 시작하지 않기 | `memo_v2.py` (O), `2memo.py` (X) |
| Python 키워드 피하기 | `memo.py` (O), `list.py` (X) — list는 Python 내장 함수 |

**클래스 이름**은 `PascalCase(CapWords)` 를 사용한다:
```python
class MemoStorage:
    pass

class UserProfile:
    pass
```

**변수와 함수 이름**은 `snake_case`:
```python
memo_count = 0

def save_memo(text):
    pass
```

**상수(변하지 않는 값)**는 `ALL_CAPS`:
```python
MAX_MEMO_COUNT = 100
DEFAULT_FILE_NAME = "memos.json"
```

---

## 2. 실제 메모 앱 폴더 구조

아무 구조 없이 시작하는 것보다, 처음부터 역할에 따라 나눠두는 것이 훨씬 편하다.

**메모 앱 폴더 구조 예시**:

```
memo_app/
├── main.py              ← 프로그램 실행 진입점
├── memo/
│   ├── __init__.py      ← 이 폴더가 패키지임을 알려주는 파일
│   ├── storage.py       ← 파일 저장/불러오기 담당
│   └── formatter.py     ← 출력 형식 담당
├── data/
│   └── memos.json       ← 실제 메모 데이터
└── tests/
    └── test_storage.py  ← 테스트 코드
```

**각 파일의 역할**:

| 파일 | 역할 |
|------|------|
| `main.py` | 프로그램 시작점. 사용자 입력을 받고 기능을 호출한다 |
| `memo/storage.py` | JSON 파일에 메모를 저장하고 불러오는 기능 |
| `memo/formatter.py` | 메모를 보기 좋게 출력하는 기능 |
| `data/memos.json` | 실제 데이터가 저장되는 파일 |
| `tests/test_storage.py` | storage.py의 기능이 올바른지 테스트 |

---

## 3. `__init__.py`가 무엇인가

`__init__.py`는 "이 폴더는 Python 패키지다"라고 Python에게 알려주는 파일이다.

이 파일이 없으면 폴더 안의 코드를 `import`할 수 없다.

```
memo/
├── __init__.py   ← 이 파일이 없으면 memo 폴더를 패키지로 인식하지 못한다
├── storage.py
└── formatter.py
```

**`__init__.py`는 내용이 비어있어도 된다**:
```python
# memo/__init__.py
# 비어있어도 괜찮다
```

**또는 자주 쓰는 것을 미리 불러와 편하게 쓸 수도 있다**:
```python
# memo/__init__.py
from .storage import load_memos, save_memos
```

이렇게 하면 외부에서 이렇게 import할 수 있다:
```python
from memo import load_memos  # memo/storage.py 안의 함수를 바로 쓸 수 있다
```

---

## 4. 절대 import vs 상대 import

### 절대 import

프로젝트 루트 디렉터리를 기준으로 전체 경로를 적는 방식이다.

```python
# main.py 에서
from memo.storage import load_memos, save_memos
from memo.formatter import display_memos
```

- 어디서 실행해도 경로가 명확하다
- 코드를 읽었을 때 어디서 가져오는지 바로 알 수 있다
- **일반적으로 절대 import를 권장한다**

### 상대 import

현재 파일을 기준으로 상대적인 위치를 적는 방식이다. `.`은 현재 패키지를 뜻한다.

```python
# memo/formatter.py 에서 같은 memo 패키지의 storage.py를 가져올 때
from .storage import load_memos

# 상위 패키지에서 가져올 때는 ..
from ..utils import helper_function
```

- 패키지 내부에서 다른 모듈을 가져올 때 사용한다
- `main.py` 같은 최상위 파일에서는 사용하면 안 된다

**언제 무엇을 쓰는가**:

| 상황 | 권장 방식 |
|------|-----------|
| `main.py`에서 하위 패키지 가져오기 | 절대 import |
| 패키지 내부 파일끼리 참조 | 상대 import 또는 절대 import 모두 가능 |
| 초보자 | 절대 import부터 익히기 |

---

## 5. `if __name__ == "__main__"` 패턴

### 왜 이 패턴이 필요한가

Python 파일은 두 가지 방식으로 실행될 수 있다.

1. **직접 실행**: `python main.py` — 이 파일이 시작점
2. **import로 불러오기**: 다른 파일에서 `import main` — 이 파일은 모듈로 사용됨

문제는 `import`를 하면 파일 안의 코드가 자동으로 실행된다는 것이다.

```python
# utils.py
print("utils.py 불러옴")  # import할 때마다 이 줄이 실행된다!

def add(a, b):
    return a + b
```

```python
# main.py
from utils import add  # "utils.py 불러옴" 이 출력되어 버린다
```

이 문제를 해결하는 것이 `if __name__ == "__main__"` 패턴이다.

### 동작 원리

Python은 파일을 실행할 때 `__name__`이라는 특수 변수를 설정한다.

- 파일을 **직접 실행**하면: `__name__ == "__main__"`
- 파일을 **import**하면: `__name__ == "파일 이름"` (예: `"utils"`)

```python
# utils.py
def add(a, b):
    return a + b

if __name__ == "__main__":
    # 이 블록은 파일을 직접 실행할 때만 실행된다
    # import할 때는 실행되지 않는다
    print(add(2, 3))
```

### 올바른 사용 예시

```python
# main.py
from memo.storage import load_memos, save_memos
from memo.formatter import display_memos

def run():
    print("메모 앱을 시작합니다.")
    memos = load_memos()
    display_memos(memos)

if __name__ == "__main__":
    run()
```

이렇게 하면:
- `python main.py` 로 직접 실행하면 `run()` 이 실행된다
- 다른 파일에서 `from main import run` 으로 import해도 `run()` 이 자동으로 실행되지 않는다

---

## 6. 실제 에러: ModuleNotFoundError

### 왜 생기는가

import하려는 모듈을 Python이 찾지 못할 때 발생한다.

```python
from memo.storage import load_memos
```
```
ModuleNotFoundError: No module named 'memo'
```

**이 에러가 나는 이유**:

1. `memo` 폴더에 `__init__.py`가 없다
2. 파일을 잘못된 위치에서 실행했다 (프로젝트 루트가 아닌 다른 폴더에서 실행)
3. 폴더 이름이나 파일 이름을 잘못 적었다

**해결 방법 1 — `__init__.py` 확인**:
```bash
# 폴더 구조 확인
memo_app/
├── main.py
└── memo/
    ├── __init__.py   ← 이 파일이 있는지 확인
    └── storage.py
```

**해결 방법 2 — 실행 위치 확인**:
```bash
# 잘못된 위치에서 실행
cd memo_app/memo
python ../main.py  # 이렇게 하면 Python이 memo 패키지를 찾지 못할 수 있다

# 올바른 실행 위치
cd memo_app
python main.py
```

---

## 7. 실제 에러: ImportError

### 왜 생기는가

모듈을 찾았지만 그 안에서 특정 이름(함수, 변수)을 가져올 수 없을 때 발생한다.

```python
from memo.storage import save_memo  # 실제 함수 이름은 save_memos
```
```
ImportError: cannot import name 'save_memo' from 'memo.storage'
```

**이 에러가 나는 이유**:
- 함수 이름을 잘못 적었다 (`save_memo` vs `save_memos`)
- 해당 함수가 그 파일에 없다
- 함수를 삭제했거나 이름을 바꿨다

**해결 방법**:
```python
# memo/storage.py 파일을 열어서 실제 함수 이름을 확인한다
# 그리고 import 구문을 정확한 이름으로 수정한다

from memo.storage import save_memos  # 's'가 있는 올바른 이름
```

---

## 실습 1 (따라 하기). 메모 앱 폴더 구조 만들기

**목표**: 실제 폴더와 파일을 만들어 메모 앱의 뼈대를 구성한다.

**1단계: 폴더 만들기**

터미널에서:
```bash
mkdir memo_app
cd memo_app
mkdir memo
mkdir data
mkdir tests
```

또는 VS Code에서 폴더를 직접 만들어도 된다.

**2단계: `__init__.py` 만들기**

`memo/__init__.py` 파일을 만들고 비워둔다 (내용 없음).

**3단계: `storage.py` 만들기**

```python
# memo/storage.py
import json
import os

DATA_FILE = "data/memos.json"

def load_memos():
    """저장된 메모 목록을 불러온다."""
    if not os.path.exists(DATA_FILE):
        return []
    with open(DATA_FILE, "r", encoding="utf-8") as file:
        return json.load(file)

def save_memos(memos):
    """메모 목록을 파일에 저장한다."""
    os.makedirs("data", exist_ok=True)  # data 폴더가 없으면 만든다
    with open(DATA_FILE, "w", encoding="utf-8") as file:
        json.dump(memos, file, ensure_ascii=False, indent=2)
```

**4단계: `formatter.py` 만들기**

```python
# memo/formatter.py

def display_memos(memos):
    """메모 목록을 보기 좋게 출력한다."""
    if not memos:
        print("저장된 메모가 없습니다.")
        return
    print("=== 메모 목록 ===")
    for i, memo in enumerate(memos, start=1):
        print(f"{i}. {memo}")
```

**5단계: `main.py` 만들기**

```python
# main.py
from memo.storage import load_memos, save_memos
from memo.formatter import display_memos

def add_memo(text):
    memos = load_memos()
    memos.append(text)
    save_memos(memos)
    print(f"추가됨: {text}")

def run():
    add_memo("with open() 패턴")
    add_memo("JSON 저장/불러오기")
    add_memo("폴더 구조 정리")
    display_memos(load_memos())

if __name__ == "__main__":
    run()
```

**실행**:
```bash
python main.py
```

**직접 해보기**: `formatter.py`에 `display_memo_count(memos)` 함수를 추가해보자. 메모가 몇 개인지 출력하는 함수다. 그리고 `main.py`에서 이 함수를 불러와 사용해보자.

---

## 실습 2 (따라 하기). `if __name__ == "__main__"` 동작 확인

**목표**: 이 패턴이 실제로 어떤 차이를 만드는지 직접 확인한다.

**`calculator.py` 만들기**:
```python
# calculator.py

def add(a, b):
    return a + b

def multiply(a, b):
    return a * b

# 이 줄은 파일을 직접 실행할 때만 출력된다
print(f"calculator.py __name__ = {__name__}")

if __name__ == "__main__":
    print("calculator.py를 직접 실행했습니다")
    print(add(3, 4))
    print(multiply(3, 4))
```

**직접 실행**:
```bash
python calculator.py
```
출력:
```
calculator.py __name__ = __main__
calculator.py를 직접 실행했습니다
7
12
```

**`main.py`에서 import**:
```python
# main.py
from calculator import add

print(f"main.py __name__ = {__name__}")
print(add(10, 20))
```
```bash
python main.py
```
출력:
```
calculator.py __name__ = calculator
main.py __name__ = __main__
30
```

`calculator.py`를 import했을 때 `"calculator.py를 직접 실행했습니다"` 는 출력되지 않는다.  
`__name__`이 `"calculator"` 이기 때문에 `if __name__ == "__main__":` 블록이 실행되지 않는다.

**직접 해보기**: `calculator.py`에서 `if __name__ == "__main__":` 을 지우고 `print("직접 실행됨")`만 남겨두면 어떻게 되는지 확인해보자.

---

## 실습 3 (따라 하기). ImportError 직접 경험하고 고치기

**오류가 있는 코드**:

```python
# main.py
from memo.storage import save_memo  # 잘못된 이름
```

**에러 메시지**:
```
ImportError: cannot import name 'save_memo' from 'memo.storage'
```

**따라 하기 — 고치는 순서**:
1. `memo/storage.py` 파일을 열어서 실제 함수 이름을 확인한다
2. `save_memos` (복수형, 's'가 있음)가 맞는 이름이다
3. import 구문을 수정한다

```python
# 수정된 main.py
from memo.storage import save_memos  # 올바른 이름
```

**직접 해보기**: `from memo.formatter import show_memos` 로 import해보자.  
`formatter.py`에 `show_memos`는 없고 `display_memos`가 있다.  
어떤 에러가 나는가? 어떻게 고쳐야 하는가?

---

## 자주 막히는 지점 (Common Pitfalls)

### Pitfall 1. `__init__.py`를 만들지 않는다

```
memo/
├── storage.py    # __init__.py가 없다
└── formatter.py

# main.py에서
from memo.storage import load_memos
# → ModuleNotFoundError: No module named 'memo'
```

해결: `memo/__init__.py` 파일을 만든다. 내용이 비어있어도 된다.

---

### Pitfall 2. 잘못된 위치에서 실행한다

```
memo_app/
├── main.py
└── memo/
    └── storage.py
```

```bash
# 잘못된 위치
cd memo_app/memo
python ../../main.py
# → ModuleNotFoundError

# 올바른 위치
cd memo_app
python main.py
```

---

### Pitfall 3. 파일 이름을 Python 내장 모듈과 같게 짓는다

```python
# list.py 라는 파일을 만들면
import list  # Python의 내장 list가 아닌 내 list.py가 import됨
# 의도치 않은 충돌 발생
```

피해야 할 파일 이름: `list.py`, `string.py`, `os.py`, `json.py`, `math.py` 등

---

### Pitfall 4. 상대 import를 최상위 파일에서 사용한다

```python
# main.py (최상위 파일)
from .memo.storage import load_memos  # 잘못된 사용
```
```
ImportError: attempted relative import with no known parent package
```

최상위 파일에서는 항상 절대 import를 사용한다.

---

### Pitfall 5. `if __name__ == "__main__":` 없이 코드를 파일 최상단에 쓴다

```python
# utils.py
def greet(name):
    return f"안녕하세요, {name}"

print(greet("테스트"))  # 이 줄이 import할 때마다 실행된다!
```

해결:
```python
# utils.py
def greet(name):
    return f"안녕하세요, {name}"

if __name__ == "__main__":
    print(greet("테스트"))  # 직접 실행할 때만 실행된다
```

---

## 확인 체크리스트

- 메모 앱의 폴더 구조를 직접 만들 수 있는가
- `__init__.py`가 왜 필요한지 설명할 수 있는가
- 절대 import와 상대 import의 차이를 말할 수 있는가
- `if __name__ == "__main__"` 이 어떤 상황에서 실행되는지 설명할 수 있는가
- `ModuleNotFoundError`와 `ImportError`의 차이를 구별할 수 있는가

---

## 한 번 더 생각해 보기

1. 모든 코드를 `main.py` 하나에 넣으면 어떤 문제가 생기는가?
2. `__init__.py`에 내용을 넣으면 어떤 편의가 생기는가?
3. `if __name__ == "__main__":` 없이 함수를 만들면 어떤 문제가 생길 수 있는가?
4. 팀 프로젝트에서 파일 이름 규칙이 왜 중요한가?

---

## 교사용 메모

- 강조: 이 장은 문법보다 "파일 역할 나누기"를 이해시키는 것이 먼저다. 메모 앱을 직접 만들어보게 하는 것이 가장 효과적이다.
- 막힘 포인트: `__init__.py` 누락으로 인한 ModuleNotFoundError, 실행 위치 오류, 상대 import 사용 실수에서 가장 많이 막힌다.
- 실습 2의 `__name__` 실험을 직접 해보게 하면 패턴의 의미를 직관적으로 이해하는 데 도움이 된다.
- 질문 1: `storage.py`에서 `if __name__ == "__main__":` 블록 안에 테스트를 넣으면 어떤 장점이 있을까?
- 질문 2: `main.py`와 `storage.py`의 역할 구분이 왜 중요한가?
