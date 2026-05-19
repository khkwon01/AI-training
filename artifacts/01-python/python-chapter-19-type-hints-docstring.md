## 이 장에서 배우는 것

- 함수에 타입 힌트(type hint)를 붙이는 방법
- 변수에 타입 힌트를 붙이는 방법
- docstring으로 함수를 설명하는 방법
- 타입 힌트가 있으면 IDE와 동료가 코드를 더 쉽게 이해한다는 사실
- 자주 쓰는 타입: `int`, `str`, `float`, `bool`, `list`, `dict`, `Optional`

---

## 먼저 쉬운 설명

코드를 처음 쓸 때는 내가 뭘 넣어야 하는지 다 기억하고 있습니다. 그런데 한 달 뒤에 그 코드를 다시 열면? "이 함수에 숫자를 넣어야 하나, 문자열을 넣어야 하나?" 하고 헷갈리기 시작합니다.

**타입 힌트**는 함수 옆에 "나는 숫자를 받아서 문자열을 돌려줘" 라고 메모를 붙여 두는 것입니다. 파이썬이 강제로 검사하지는 않지만, VS Code 같은 편집기가 즉시 알려줍니다.

**docstring**은 함수 안에 쓰는 설명서입니다. `help(함수이름)`을 실행하면 바로 이 설명이 나타납니다.

둘 다 코드를 *실행하는 데* 영향을 주지 않습니다. 하지만 코드를 *읽고 유지하는 데* 엄청난 차이를 만듭니다.

---

## 1. 함수에 타입 힌트 붙이기

타입 힌트는 매개변수 이름 뒤에 `: 타입`, 반환값은 `) -> 타입:` 형식으로 씁니다.

```python
# 파일: calculator.py

# 타입 힌트 없음 — 뭘 넣어야 할지 모름
def add(a, b):
    return a + b


# 타입 힌트 있음 — int 두 개를 받아서 int를 돌려줌
def add(a: int, b: int) -> int:
    return a + b


# 문자열을 받아서 문자열을 돌려주는 함수
def greet(name: str) -> str:
    return f"안녕하세요, {name}님!"


# 아무것도 돌려주지 않을 때는 -> None
def print_score(score: int) -> None:
    print(f"점수: {score}점")
```

> **핵심 규칙**: 반환값이 없으면 `-> None`을 씁니다. 아예 생략해도 되지만, 명시적으로 쓰면 더 명확합니다.

---

## 2. 변수에 타입 힌트 붙이기

함수 바깥의 변수에도 타입 힌트를 붙일 수 있습니다. 형식은 `변수명: 타입 = 값`입니다.

```python
# 파일: student.py

# 일반 변수
student_name: str = "김민준"
student_age: int = 20
average_score: float = 87.5
is_passed: bool = True

# 리스트와 딕셔너리 (파이썬 3.9 이상)
scores: list[int] = [85, 90, 78]
grade_map: dict[str, int] = {"수학": 90, "영어": 85}

# 값이 없을 수도 있을 때 — Optional 사용 (파이썬 3.9 이하)
from typing import Optional

nickname: Optional[str] = None   # str이거나 None
nickname = "민준이"               # 나중에 값을 넣어도 됨
```

> **파이썬 버전 주의**: `list[int]`, `dict[str, int]`처럼 소문자로 쓰는 방식은 파이썬 **3.9 이상**에서 됩니다. 3.8 이하라면 `from typing import List, Dict`를 임포트해서 `List[int]`, `Dict[str, int]`로 씁니다.

---

## 3. docstring으로 함수 설명하기

docstring은 함수 첫 줄에 `"""큰따옴표 세 개"""` 로 씁니다. 한 줄짜리와 여러 줄짜리 두 가지 형태가 있습니다.

```python
# 파일: calculator.py

def add(a: int, b: int) -> int:
    """두 정수를 더한 결과를 반환합니다."""
    return a + b


def calculate_bmi(weight: float, height: float) -> float:
    """
    BMI(체질량지수)를 계산합니다.

    Args:
        weight: 몸무게 (킬로그램 단위)
        height: 키 (미터 단위, 예: 1.75)

    Returns:
        BMI 값 (소수점 첫째 자리까지)

    Examples:
        >>> calculate_bmi(70, 1.75)
        22.9
    """
    bmi = weight / (height ** 2)
    return round(bmi, 1)
```

docstring을 확인하는 방법:

```python
# 터미널 또는 REPL에서
help(calculate_bmi)
print(calculate_bmi.__doc__)
```

출력:

```
BMI(체질량지수)를 계산합니다.

Args:
    weight: 몸무게 (킬로그램 단위)
    height: 키 (미터 단위, 예: 1.75)
...
```

---

## 4. 실전 예시 — 학생 성적 관리 함수

타입 힌트와 docstring을 함께 쓰는 완성된 예시입니다.

```python
# 파일: grade_manager.py

from typing import Optional


def get_letter_grade(score: int) -> str:
    """
    점수를 받아서 등급 문자를 반환합니다.

    Args:
        score: 0~100 사이의 정수 점수

    Returns:
        'A', 'B', 'C', 'D', 'F' 중 하나

    Examples:
        >>> get_letter_grade(95)
        'A'
        >>> get_letter_grade(72)
        'C'
    """
    if score >= 90:
        return "A"
    elif score >= 80:
        return "B"
    elif score >= 70:
        return "C"
    elif score >= 60:
        return "D"
    else:
        return "F"


def summarize_scores(scores: list[int]) -> dict[str, float]:
    """
    점수 목록을 받아서 평균, 최고점, 최저점을 반환합니다.

    Args:
        scores: 정수 점수 목록 (비어 있으면 안 됨)

    Returns:
        {'average': 평균, 'highest': 최고점, 'lowest': 최저점}
    """
    return {
        "average": sum(scores) / len(scores),
        "highest": float(max(scores)),
        "lowest": float(min(scores)),
    }


def find_student(
    name: str,
    students: list[str],
) -> Optional[int]:
    """
    학생 이름으로 목록에서 인덱스를 찾습니다.

    Args:
        name: 찾을 학생 이름
        students: 학생 이름 목록

    Returns:
        찾으면 인덱스(int), 없으면 None
    """
    if name in students:
        return students.index(name)
    return None
```

---

## 따라 하기 실습

### 실습 1 — 기존 함수에 타입 힌트 추가하기

`practice_hints.py` 파일을 만들고 아래 코드를 붙여 넣으세요. 그런 다음 `???` 부분을 채워서 타입 힌트를 완성하세요.

```python
# 파일: practice_hints.py

# TODO: ??? 자리에 알맞은 타입을 채우세요

def multiply(x: ???, y: ???) -> ???:
    return x * y


def is_adult(age: ???) -> ???:
    return age >= 18


def full_name(first: ???, last: ???) -> ???:
    return f"{last} {first}"
```

완성된 코드를 실행해서 오류가 없으면 성공입니다:

```python
print(multiply(3, 4))       # 12
print(is_adult(20))         # True
print(full_name("민준", "김"))  # 김 민준
```

---

### 실습 2 — docstring 작성하기

실습 1에서 완성한 `practice_hints.py`에 각 함수마다 docstring을 추가하세요. `multiply` 예시를 참고하세요.

```python
def multiply(x: int, y: int) -> int:
    """
    두 정수를 곱한 결과를 반환합니다.

    Args:
        x: 첫 번째 정수
        y: 두 번째 정수

    Returns:
        x와 y를 곱한 정수
    """
    return x * y
```

작성 후 터미널에서 확인하세요:

```python
python3 -c "from practice_hints import is_adult; help(is_adult)"
```

---

### 실습 3 — 처음부터 타입 힌트와 docstring으로 함수 만들기

`temperature.py` 파일을 새로 만들고, 아래 두 함수를 타입 힌트와 docstring을 포함해서 직접 작성하세요.

- `celsius_to_fahrenheit(celsius: float) -> float` — 섭씨를 화씨로 변환 (공식: `(celsius * 9/5) + 32`)
- `is_freezing(celsius: float) -> bool` — 섭씨 온도가 0도 이하이면 `True`

완성 후 아래로 검증:

```python
# temperature.py 맨 아래에 추가
if __name__ == "__main__":
    print(celsius_to_fahrenheit(0))    # 32.0
    print(celsius_to_fahrenheit(100))  # 212.0
    print(is_freezing(-5))             # True
    print(is_freezing(20))             # False
```

---

## 자주 하는 실수

| 실수 | 오류 메시지 또는 증상 | 올바른 방법 |
|------|----------------------|-------------|
| 콜론 대신 등호 사용 | `SyntaxError: invalid syntax` | `def f(x: int)` — `=` 아니라 `:` |
| 화살표를 `->`가 아닌 `=>` 로 씀 | `SyntaxError: invalid syntax` | `-> int:` 로 씀 |
| 반환 타입 힌트 뒤에 콜론 빠뜨림 | `SyntaxError: expected ':'` | `def f() -> int:` 마지막 `:` 필수 |
| 파이썬 3.8에서 `list[int]` 사용 | `TypeError: 'type' object is not subscriptable` | `from typing import List` 후 `List[int]` 사용 |
| docstring을 `#` 주석으로 씀 | `help()`에 설명이 안 나옴 | 함수 첫 줄에 `"""설명"""` 으로 써야 함 |
| `Optional` 없이 `None` 반환 타입 표현 | `mypy` 경고 또는 IDE 경고 | `from typing import Optional` 후 `Optional[str]` 사용 |
| 타입 힌트를 쓰면 파이썬이 강제로 검사한다고 오해 | 런타임에 오류 없이 그냥 실행됨 | 힌트는 메모일 뿐, 강제 검사는 `mypy` 같은 별도 도구 필요 |

---

## 확인 체크리스트

- [ ] 함수 매개변수에 `: 타입` 형식으로 타입 힌트를 붙일 수 있다
- [ ] 반환값에 `-> 타입` 형식으로 타입 힌트를 붙일 수 있다
- [ ] 반환값이 없는 함수에 `-> None`을 쓸 수 있다
- [ ] 변수에 `변수명: 타입 = 값` 형식으로 타입 힌트를 붙일 수 있다
- [ ] `list[int]`, `dict[str, int]` 같은 컬렉션 타입 힌트를 쓸 수 있다
- [ ] 값이 `None`일 수도 있을 때 `Optional[타입]`을 쓸 수 있다
- [ ] 함수 첫 줄에 `"""..."""` 으로 docstring을 쓸 수 있다
- [ ] `Args:`, `Returns:` 섹션을 포함한 여러 줄 docstring을 쓸 수 있다
- [ ] `help(함수이름)`으로 docstring을 확인할 수 있다
- [ ] 타입 힌트는 런타임 강제가 아니라 메모임을 이해한다

---

## 한 번 더 생각해 보기

1. 타입 힌트를 쓰면 파이썬이 자동으로 오류를 막아 줄까요? 직접 `def add(a: int, b: int) -> int: return a + b` 를 정의한 뒤 `add("hello", "world")`를 실행해 보고, 결과를 확인한 뒤 왜 그런지 설명해 보세요.

2. 함수가 "학생을 찾으면 이름을 돌려주고, 없으면 아무것도 안 돌려줘야" 할 때 반환 타입 힌트를 어떻게 써야 할까요? `Optional`을 쓰는 방법과 쓰지 않는 방법을 각각 생각해 보세요.

3. 팀 프로젝트에서 내가 만든 함수를 동료가 쓴다면, 타입 힌트와 docstring 중 어느 것이 더 중요할까요? 아니면 둘 다 필요할까요? 이유를 말해 보세요.

---

## 다음 장

다음 장에서는 타입 힌트를 자동으로 검사해 주는 도구 **mypy**와 코드 스타일을 잡아 주는 **ruff**를 설치하고 사용하는 방법을 배웁니다.