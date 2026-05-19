## 이 장에서 배우는 것

- 테스트가 무엇인지, 왜 코드를 짤 때 함께 써야 하는지 이해한다
- `pytest`를 설치하고 처음으로 테스트를 실행할 수 있다
- `assert` 문으로 코드가 원하는 대로 동작하는지 확인할 수 있다
- 테스트 파일과 테스트 함수의 이름 규칙을 지킬 수 있다
- 테스트가 실패했을 때 메시지를 읽고 문제를 찾을 수 있다
- 여러 경우를 한 번에 테스트하는 `parametrize`를 사용할 수 있다

---

## 먼저 쉬운 설명

요리사가 새 메뉴를 손님에게 내기 전에 먼저 맛을 보는 것처럼, 프로그래머도 코드를 다른 사람이 쓰기 전에 "이 코드가 제대로 동작하나?" 확인해야 합니다.

그런데 매번 직접 코드를 실행해서 눈으로 확인하면 어떨까요? 코드가 길어질수록 확인해야 할 것도 많아지고, 어제 잘 되던 기능이 오늘 갑자기 망가져도 눈치채기 힘들어집니다.

**테스트 코드**는 이 확인 작업을 자동으로 해 주는 코드입니다. 버튼 하나로 "내 코드의 모든 부분이 여전히 잘 동작하는지" 1초 만에 검사할 수 있습니다. 처음에는 테스트 코드를 짜는 게 일이 두 배가 되는 것처럼 느껴지지만, 나중에 코드를 고칠 때 훨씬 빠르고 안전하게 작업할 수 있게 해 줍니다.

이 장에서는 Python에서 가장 많이 쓰이는 테스트 도구인 `pytest`를 배웁니다.

---

## 1. pytest 설치와 첫 실행

### pytest 설치

터미널(또는 VS Code 터미널)에서 아래 명령어를 실행하세요.

```bash
pip install pytest
```

설치가 잘 됐는지 확인합니다.

```bash
pytest --version
```

```
pytest 8.x.x
```

### 폴더 구조 준비

이 장에서 만들 파일 구조입니다. 먼저 `calculator` 폴더를 만들고 시작하겠습니다.

```
calculator/
├── calc.py          ← 실제 기능 코드
└── test_calc.py     ← 테스트 코드
```

### 첫 번째 함수와 테스트 작성

**`calc.py`**

```python
def add(a, b):
    return a + b
```

**`test_calc.py`**

```python
from calc import add

def test_add_두_양수():
    결과 = add(3, 5)
    assert 결과 == 8

def test_add_음수포함():
    결과 = add(-1, 4)
    assert 결과 == 3
```

### 테스트 실행

`calculator` 폴더 안에서 아래를 실행합니다.

```bash
pytest test_calc.py
```

성공하면 이렇게 나옵니다.

```
collected 2 items

test_calc.py ..                    [100%]

2 passed in 0.03s
```

점(`.`) 하나가 테스트 하나 통과를 뜻합니다.

> **이름 규칙 반드시 지키기**
> - 테스트 파일 이름은 반드시 `test_`로 시작하거나 `_test.py`로 끝나야 합니다.
> - 테스트 함수 이름은 반드시 `test_`로 시작해야 합니다.
> - 이 규칙을 어기면 `pytest`가 테스트를 찾지 못합니다.

---

## 2. assert로 결과 확인하기

`assert`는 "이 조건이 참이어야 한다"고 선언하는 Python 키워드입니다. 조건이 거짓이면 테스트가 실패합니다.

### 자주 쓰는 assert 패턴

**`calc.py`** (기능 추가)

```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        raise ValueError("0으로 나눌 수 없습니다.")
    return a / b

def is_even(n):
    return n % 2 == 0
```

**`test_calc.py`** (다양한 assert 예시)

```python
from calc import add, subtract, multiply, divide, is_even

# 숫자가 같은지 확인
def test_subtract():
    assert subtract(10, 3) == 7

# 참/거짓 확인
def test_is_even_짝수():
    assert is_even(4) is True

def test_is_even_홀수():
    assert is_even(7) is False

# 리스트, 문자열도 비교 가능
def test_결과_타입_확인():
    결과 = divide(10, 4)
    assert isinstance(결과, float)

# 예외(에러)가 발생하는지 확인
def test_divide_0으로나누기():
    import pytest
    with pytest.raises(ValueError) as exc_info:
        divide(5, 0)
    assert "0으로 나눌 수 없습니다" in str(exc_info.value)
```

### 테스트가 실패하면 어떤 메시지가 나올까?

`add` 함수를 일부러 잘못 만들어 봅시다.

```python
def add(a, b):
    return a - b  # 버그! 빼기로 잘못 작성
```

```bash
pytest test_calc.py::test_add_두_양수
```

```
FAILED test_calc.py::test_add_두_양수 - AssertionError: assert -2 == 8
```

pytest는 어느 파일, 어느 함수, 어떤 값이 잘못됐는지 정확하게 알려줍니다.

---

## 3. parametrize로 여러 경우 한 번에 테스트하기

같은 함수를 여러 입력값으로 테스트하고 싶을 때 `@pytest.mark.parametrize`를 쓰면 코드 반복 없이 깔끔하게 작성할 수 있습니다.

### parametrize 없이 작성한 경우 (반복이 많음)

```python
def test_add_case1():
    assert add(1, 2) == 3

def test_add_case2():
    assert add(0, 0) == 0

def test_add_case3():
    assert add(-5, 5) == 0
```

### parametrize 사용 (권장)

**`test_calc.py`** (아래 코드 추가)

```python
import pytest
from calc import add, multiply

@pytest.mark.parametrize("a, b, 기댓값", [
    (1, 2, 3),
    (0, 0, 0),
    (-5, 5, 0),
    (100, -50, 50),
])
def test_add_여러경우(a, b, 기댓값):
    assert add(a, b) == 기댓값


@pytest.mark.parametrize("a, b, 기댓값", [
    (3, 4, 12),
    (0, 100, 0),
    (-2, -3, 6),
])
def test_multiply_여러경우(a, b, 기댓값):
    assert multiply(a, b) == 기댓값
```

실행 결과:

```
test_calc.py::test_add_여러경우[1-2-3] PASSED
test_calc.py::test_add_여러경우[0-0-0] PASSED
test_calc.py::test_add_여러경우[-5-5-0] PASSED
test_calc.py::test_add_여러경우[100--50-50] PASSED
```

각 케이스가 독립적으로 실행되기 때문에, 하나가 실패해도 나머지 결과를 모두 확인할 수 있습니다.

---

## 4. 테스트 구조: Arrange-Act-Assert 패턴

좋은 테스트는 세 단계로 구분해서 작성합니다. 이 구조를 **AAA 패턴**이라고 부릅니다.

```
Arrange  →  준비 (입력값, 초기 상태 설정)
Act      →  실행 (테스트할 함수 호출)
Assert   →  확인 (결과가 기댓값과 같은지 검증)
```

### 예시: 학생 점수 계산기

**`grade.py`**

```python
def calculate_grade(점수):
    """
    점수를 받아서 학점 문자열을 반환한다.
    90 이상: 'A', 80 이상: 'B', 70 이상: 'C', 그 외: 'F'
    """
    if 점수 >= 90:
        return 'A'
    elif 점수 >= 80:
        return 'B'
    elif 점수 >= 70:
        return 'C'
    else:
        return 'F'

def average(점수_리스트):
    if not 점수_리스트:
        raise ValueError("점수 리스트가 비어 있습니다.")
    return sum(점수_리스트) / len(점수_리스트)
```

**`test_grade.py`**

```python
from grade import calculate_grade, average
import pytest

def test_calculate_grade_A학점():
    # Arrange
    입력_점수 = 95

    # Act
    결과 = calculate_grade(입력_점수)

    # Assert
    assert 결과 == 'A'

def test_calculate_grade_경계값_B():
    # 80점은 B학점이어야 한다 (경계값 테스트)
    assert calculate_grade(80) == 'B'

def test_average_정상():
    assert average([70, 80, 90]) == 80.0

def test_average_빈_리스트_예외():
    with pytest.raises(ValueError) as exc_info:
        average([])
    assert "비어 있습니다" in str(exc_info.value)
```

---

## 따라 하기 실습

### 실습 1 — 문자열 유틸리티 함수 테스트 작성하기

다음 파일을 그대로 만들고 테스트를 실행해 보세요.

**`string_utils.py`** 생성:

```python
def reverse_string(s):
    return s[::-1]

def count_vowels(s):
    모음 = "aeiouAEIOU"
    return sum(1 for c in s if c in 모음)

def is_palindrome(s):
    s = s.lower().replace(" ", "")
    return s == s[::-1]
```

**`test_string_utils.py`** 생성:

```python
from string_utils import reverse_string, count_vowels, is_palindrome
import pytest

def test_reverse_string_일반():
    assert reverse_string("hello") == "olleh"

def test_reverse_string_빈문자열():
    assert reverse_string("") == ""

def test_count_vowels():
    assert count_vowels("hello") == 2

@pytest.mark.parametrize("단어, 기댓값", [
    ("racecar", True),
    ("level", True),
    ("hello", False),
    ("A man a plan a canal Panama", True),
])
def test_is_palindrome(단어, 기댓값):
    assert is_palindrome(단어) == 기댓값
```

터미널에서 실행:

```bash
pytest test_string_utils.py -v
```

`-v` 옵션을 붙이면 각 테스트 이름이 자세히 출력됩니다.

---

### 실습 2 — 버그 찾기 실습

아래 `buggy_calc.py`에는 버그가 하나 숨어 있습니다. 테스트를 작성해서 버그를 찾고 수정해 보세요.

**`buggy_calc.py`** 생성:

```python
def celsius_to_fahrenheit(celsius):
    # 섭씨 → 화씨 변환: F = C × 9/5 + 32
    return celsius * 9 / 5 + 3  # ← 버그가 있습니다!

def fahrenheit_to_celsius(fahrenheit):
    return (fahrenheit - 32) * 5 / 9
```

**`test_buggy_calc.py`** 생성 후 직접 작성해 보세요:

```python
from buggy_calc import celsius_to_fahrenheit, fahrenheit_to_celsius
import pytest

def test_섭씨0도는_화씨32도():
    # 0°C = 32°F
    assert celsius_to_fahrenheit(0) == 32  # 이 테스트가 실패할 것입니다!

def test_섭씨100도는_화씨212도():
    assert celsius_to_fahrenheit(100) == 212

@pytest.mark.parametrize("화씨, 기댓_섭씨", [
    (32, 0),
    (212, 100),
    (98.6, 37),
])
def test_fahrenheit_to_celsius(화씨, 기댓_섭씨):
    결과 = fahrenheit_to_celsius(화씨)
    assert round(결과, 1) == 기댓_섭씨
```

테스트를 실행하면 버그가 어디 있는지 바로 알 수 있습니다. `buggy_calc.py`의 `+ 3`을 `+ 32`로 고친 뒤 다시 실행해 보세요.

---

### 실습 3 — 장바구니 기능 테스트 작성하기

이번엔 조금 더 현실적인 시나리오입니다. 아래 코드를 만들고 TODO 부분을 직접 완성해 보세요.

**`cart.py`** 생성:

```python
class Cart:
    def __init__(self):
        self.items = {}  # {상품명: (가격, 수량)}

    def add_item(self, 상품명, 가격, 수량=1):
        if 상품명 in self.items:
            기존_가격, 기존_수량 = self.items[상품명]
            self.items[상품명] = (기존_가격, 기존_수량 + 수량)
        else:
            self.items[상품명] = (가격, 수량)

    def remove_item(self, 상품명):
        if 상품명 not in self.items:
            raise KeyError(f"'{상품명}' 상품이 장바구니에 없습니다.")
        del self.items[상품명]

    def total_price(self):
        return sum(가격 * 수량 for 가격, 수량 in self.items.values())
```

**`test_cart.py`** 생성:

```python
from cart import Cart
import pytest

def test_상품_추가_후_총가격():
    # Arrange
    장바구니 = Cart()

    # Act
    장바구니.add_item("사과", 1000, 3)
    장바구니.add_item("바나나", 500, 2)

    # Assert
    assert 장바구니.total_price() == 4000  # 1000*3 + 500*2

def test_같은_상품_두번_추가_수량누적():
    장바구니 = Cart()
    장바구니.add_item("우유", 1500, 1)
    장바구니.add_item("우유", 1500, 2)
    _, 수량 = 장바구니.items["우유"]
    assert 수량 == 3

def test_없는_상품_삭제시_예외():
    장바구니 = Cart()
    with pytest.raises(KeyError) as exc_info:
        장바구니.remove_item("존재하지않는상품")
    assert "존재하지않는상품" in str(exc_info.value)

def test_빈_장바구니_총가격은_0():
    # TODO: 빈 Cart를 만들고 total_price()가 0인지 확인하세요
    pass  # ← 이 부분을 완성해 보세요

def test_상품_삭제_후_총가격():
    # TODO: 상품을 추가하고 삭제한 뒤 총가격을 확인하세요
    pass  # ← 이 부분을 완성해 보세요
```

---

## 자주 하는 실수

| 실수 | 실제 에러 메시지 | 올바른 방법 |
|------|----------------|------------|
| 테스트 파일 이름이 `test_`로 시작 안 함 | `collected 0 items` — pytest가 아무것도 실행 안 함 | 파일명을 `test_calc.py` 형식으로 변경 |
| 테스트 함수 이름이 `test_`로 시작 안 함 | `collected 0 items` — 마찬가지로 실행 안 됨 | 함수명을 `def test_add():` 형식으로 변경 |
| `assert` 대신 `print`로 결과 확인 | 테스트가 항상 통과됨 (버그가 있어도!) | 반드시 `assert 결과 == 기댓값` 형식 사용 |
| `pytest.raises` 없이 예외 테스트 | `FAILED ... - ZeroDivisionError` — 테스트 자체가 터짐 | `with pytest.raises(예외타입):` 블록 사용 |
| import 경로 실수 | `ModuleNotFoundError: No module named 'calc'` | 테스트 파일과 소스 파일이 같은 폴더에 있는지 확인 |
| `assert add(1, 2) = 3` (등호 하나) | `SyntaxError: invalid syntax` | `==` (등호 두 개) 사용 |
| 부동소수점 비교 | `AssertionError: assert 0.30000000000000004 == 0.3` | `assert abs(결과 - 0.3) < 1e-9` 또는 `pytest.approx(0.3)` 사용 |

### 부동소수점 비교 올바르게 하기

```python
# 잘못된 방법
def test_divide_부동소수점_잘못():
    assert 0.1 + 0.2 == 0.3  # 항상 실패!

# 올바른 방법 1: pytest.approx 사용
import pytest

def test_divide_부동소수점_올바른():
    assert 0.1 + 0.2 == pytest.approx(0.3)
```

---

## 확인 체크리스트

스스로 아래 항목을 하나씩 체크해 보세요.

- [ ] `pip install pytest` 명령어로 pytest를 설치할 수 있다
- [ ] `pytest --version`으로 설치 여부를 확인할 수 있다
- [ ] 테스트 파일은 `test_`로 시작해야 한다는 규칙을 안다
- [ ] 테스트 함수는 `def test_`로 시작해야 한다는 규칙을 안다
- [ ] `assert 결과 == 기댓값` 형태로 테스트를 검증할 수 있다
- [ ] `pytest test_파일명.py` 명령어로 테스트를 실행할 수 있다
- [ ] `pytest test_파일명.py -v`로 자세한 결과를 볼 수 있다
- [ ] 테스트 실패 메시지를 읽고 어떤 값이 틀렸는지 찾을 수 있다
- [ ] `pytest.raises(예외타입)`으로 예외 발생을 테스트할 수 있다
- [ ] `@pytest.mark.parametrize`로 여러 케이스를 한 번에 테스트할 수 있다
- [ ] AAA 패턴(Arrange-Act-Assert)이 무엇인지 설명할 수 있다
- [ ] 실습 1, 2, 3의 TODO를 모두 완성하고 전체 테스트가 통과한다

---

## 한 번 더 생각해 보기

1. **"테스트 코드도 코드다"** — 만약 테스트 코드 자체에 버그가 있다면 어떤 문제가 생길까요? 테스트가 항상 통과하는데 사실은 아무것도 검증을 안 하는 상황이 생길 수 있을까요? 실습 2에서 `assert celsius_to_fahrenheit(0) == 0`으로 잘못 작성했다면 버그를 찾을 수 있었을까요?

2. **경계값의 중요성** — `calculate_grade` 함수에서 79점, 80점, 81점 각각의 결과가 다릅니다. 왜 경계값(80점, 90점)을 특별히 테스트해야 할까요? 개발자가 `점수 >= 80` 대신 `점수 > 80`으로 실수로 작성했다면, 어떤 테스트가 이 버그를 잡아낼 수 있을까요?

3. **테스트를 먼저 작성하는 방식** — 코드를 짜기 전에 테스트를 먼저 작성하는 개발 방식을 **TDD(테스트 주도 개발, Test-Driven Development)**라고 합니다. `cart.py`에서 `remove_item` 함수를 구현하기 전에 `test_상품_삭제_후_총가격` 테스트를 먼저 작성했다면 어떤 장점이 있었을까요?

---

## 다음 장

다음 장에서는 여러 테스트에서 반복되는 준비 코드를 깔끔하게 정리하는 `fixture`와, 외부 API나 데이터베이스 없이도 테스트할 수 있게 해 주는 `mock` 사용법을 배웁니다.