# Chapter 23: AI 문서화 자동화

## 이 장에서 배우는 것

- AI 도구(Claude, GitHub Copilot 등)를 사용해 코드 문서를 자동으로 작성하는 방법
- Python docstring의 기본 형식과 작성 규칙
- 함수, 클래스, 모듈 수준의 문서화 방법
- AI가 생성한 문서를 직접 검토하고 수정하는 방법
- `pydoc`과 `help()` 명령어로 문서를 확인하는 방법

---

## 먼저 쉬운 설명

코드를 처음 작성할 때는 "내가 나중에도 이 코드를 이해하겠지"라고 생각합니다. 하지만 3개월 후에 같은 코드를 열면 무슨 일이 일어날까요?

```
# 이게 뭐였더라...?
def calc(x, y, z=True):
    return x * y if z else x + y
```

이런 코드를 보면 멍해집니다. 함수 이름도 짧고, 파라미터 이름도 불명확하고, 주석도 없습니다.

**문서화(documentation)**는 코드가 *무엇을 하는지*, *어떻게 사용하는지*를 설명하는 글입니다. 예전에는 이 문서를 전부 손으로 써야 했습니다. 이제는 AI에게 초안을 맡기고 사람이 검토·수정하는 방식으로 훨씬 빠르게 작업할 수 있습니다.

이 장에서는 AI를 활용해 실제 프로젝트 수준의 문서를 만드는 전체 흐름을 배웁니다.

---

## 1. Python docstring이란 무엇인가

함수나 클래스 바로 아래에 `"""삼중 따옴표"""`로 작성하는 문자열을 **docstring**이라고 합니다. Python 자체가 이 문자열을 `__doc__` 속성으로 저장하기 때문에 `help()` 명령어나 IDE의 자동완성에 자동으로 표시됩니다.

**파일: `calculator.py`**

```python
def add(a: int, b: int) -> int:
    """두 정수를 더해 결과를 반환합니다."""
    return a + b


def divide(a: float, b: float) -> float:
    """
    두 숫자를 나눕니다.

    b가 0이면 ZeroDivisionError가 발생합니다.
    """
    return a / b
```

Python 인터프리터에서 확인하는 방법:

```python
>>> from calculator import add
>>> help(add)
Help on function add in module calculator:

add(a: int, b: int) -> int
    두 정수를 더해 결과를 반환합니다.
```

---

## 2. Google 스타일 docstring — AI가 가장 잘 생성하는 형식

docstring에는 여러 형식이 있지만, **Google 스타일**이 가장 읽기 쉽고 AI 도구도 가장 자주 이 형식으로 출력합니다.

**파일: `user_service.py`**

```python
def create_user(name: str, age: int, email: str) -> dict:
    """
    새 사용자를 생성하고 딕셔너리로 반환합니다.

    Args:
        name: 사용자의 실제 이름. 빈 문자열은 허용되지 않습니다.
        age: 사용자 나이. 0 이상의 정수여야 합니다.
        email: 로그인에 사용할 이메일 주소.

    Returns:
        생성된 사용자 정보를 담은 딕셔너리.
        예: {"name": "홍길동", "age": 30, "email": "hong@example.com"}

    Raises:
        ValueError: name이 빈 문자열이거나 age가 음수일 때.

    Example:
        >>> user = create_user("홍길동", 30, "hong@example.com")
        >>> print(user["name"])
        홍길동
    """
    if not name:
        raise ValueError("이름은 빈 문자열일 수 없습니다.")
    if age < 0:
        raise ValueError(f"나이는 0 이상이어야 합니다. 입력값: {age}")
    return {"name": name, "age": age, "email": email}
```

각 섹션의 역할:

| 섹션 | 설명 |
|------|------|
| 첫 줄 | 한 문장 요약 — 가장 중요 |
| `Args` | 파라미터 이름, 타입, 의미 |
| `Returns` | 반환값의 형태와 의미 |
| `Raises` | 발생 가능한 예외와 조건 |
| `Example` | 실제 사용 예시 (복사해서 바로 실행 가능해야 함) |

---

## 3. AI에게 docstring 초안을 요청하는 방법

AI에게 문서 초안을 맡길 때 **좋은 프롬프트**와 **나쁜 프롬프트**의 차이는 큽니다.

**나쁜 프롬프트 예시:**
```
이 코드 문서화해줘
```

**좋은 프롬프트 예시:**
```
아래 Python 함수에 Google 스타일 docstring을 한국어로 작성해줘.
Args, Returns, Raises, Example 섹션을 모두 포함하고,
Example은 실제로 실행 가능한 코드로 써줘.

def send_email(to: str, subject: str, body: str, cc: list = None) -> bool:
    ...
```

AI가 생성한 초안의 예:

**파일: `email_service.py`**

```python
def send_email(
    to: str,
    subject: str,
    body: str,
    cc: list = None,
) -> bool:
    """
    지정한 수신자에게 이메일을 발송합니다.

    Args:
        to: 수신자 이메일 주소. 올바른 형식이어야 합니다.
        subject: 이메일 제목. 빈 문자열도 허용됩니다.
        body: 이메일 본문 내용.
        cc: 참조(CC)에 추가할 이메일 주소 목록. 기본값은 None (참조 없음).

    Returns:
        발송 성공 시 True, 실패 시 False를 반환합니다.

    Raises:
        ValueError: to가 올바른 이메일 형식이 아닐 때.
        ConnectionError: 메일 서버에 연결할 수 없을 때.

    Example:
        >>> success = send_email(
        ...     to="recipient@example.com",
        ...     subject="안녕하세요",
        ...     body="반갑습니다.",
        ...     cc=["manager@example.com"],
        ... )
        >>> print(success)
        True
    """
    # 실제 구현은 여기에
    ...
```

> **AI 문서를 그대로 믿지 마세요.** AI는 함수 내부 로직을 직접 실행하지 않기 때문에 `Raises` 섹션이나 `Example` 출력값이 틀릴 수 있습니다. 반드시 직접 실행해서 확인하세요.

---

## 4. 클래스 문서화

클래스는 클래스 자체와 `__init__` 메서드 두 곳에 docstring을 씁니다. Google 스타일에서는 `__init__`의 파라미터를 클래스 docstring의 `Attributes` 섹션에 모아서 씁니다.

**파일: `order.py`**

```python
class Order:
    """
    고객의 주문 정보를 표현하는 클래스입니다.

    Attributes:
        order_id: 고유한 주문 번호 (문자열).
        items: 주문된 상품 이름 목록.
        total_price: 총 주문 금액 (원 단위).
        is_paid: 결제 완료 여부.

    Example:
        >>> order = Order("ORD-001", ["노트북", "마우스"], 1500000)
        >>> order.add_item("키보드", 80000)
        >>> print(order.total_price)
        1580000
    """

    def __init__(self, order_id: str, items: list, total_price: int) -> None:
        self.order_id = order_id
        self.items = items
        self.total_price = total_price
        self.is_paid = False

    def add_item(self, item_name: str, price: int) -> None:
        """
        주문에 상품을 추가합니다.

        Args:
            item_name: 추가할 상품 이름.
            price: 상품 가격 (원 단위, 0 이상).

        Raises:
            ValueError: price가 음수일 때.
        """
        if price < 0:
            raise ValueError(f"가격은 음수일 수 없습니다: {price}")
        self.items.append(item_name)
        self.total_price += price
```

---

## 5. 모듈 수준 docstring

파이썬 파일 맨 위에도 docstring을 쓸 수 있습니다. 이 파일이 무엇을 하는 모듈인지 한눈에 알 수 있게 해줍니다.

**파일: `report_generator.py`**

```python
"""
월별 판매 보고서를 생성하는 유틸리티 모듈입니다.

이 모듈은 데이터베이스에서 판매 데이터를 가져와
PDF 또는 CSV 형식의 보고서를 만드는 함수를 제공합니다.

Typical usage example::

    from report_generator import generate_monthly_report

    report_path = generate_monthly_report(year=2026, month=5)
    print(f"보고서가 생성되었습니다: {report_path}")
"""

import datetime
from pathlib import Path
```

---

## 따라 하기 실습

### 실습 1 — 문서 없는 함수에 AI로 docstring 추가하기

아래 코드를 `password_utils.py` 파일로 저장하세요.

```python
# 파일: password_utils.py

def check_password_strength(password: str) -> str:
    score = 0
    if len(password) >= 8:
        score += 1
    if any(c.isupper() for c in password):
        score += 1
    if any(c.isdigit() for c in password):
        score += 1
    if any(c in "!@#$%^&*" for c in password):
        score += 1

    if score <= 1:
        return "약함"
    elif score == 2:
        return "보통"
    elif score == 3:
        return "강함"
    else:
        return "매우 강함"
```

AI(Claude 또는 Copilot)에게 다음 프롬프트로 docstring을 요청하세요:

```
위 check_password_strength 함수에 Google 스타일 docstring을 한국어로 작성해줘.
각 점수 조건(대문자, 숫자, 특수문자)도 Args 또는 설명 부분에 포함해줘.
Example 섹션에 최소 3가지 케이스를 넣어줘.
```

AI가 생성한 docstring을 함수에 붙여넣고, 터미널에서 확인합니다:

```bash
python3 -c "from password_utils import check_password_strength; help(check_password_strength)"
```

### 실습 2 — Example 섹션 검증하기

실습 1에서 AI가 생성한 `Example` 섹션의 출력값이 실제와 일치하는지 확인합니다.

```bash
python3 -m doctest password_utils.py -v
```

오류가 나면 예시를 수정합니다. 흔한 문제:

```
Failed example:
    check_password_strength("abc")
Expected:
    '약함'
Got:
    '약함'

# 따옴표 종류(홑따옴표 vs 쌍따옴표)가 다를 수 있습니다.
```

`Expected` 줄의 따옴표를 실제 출력에 맞게 수정하면 됩니다.

### 실습 3 — 클래스 문서화 및 전체 모듈 검토

`password_utils.py`에 클래스를 추가하고 문서화합니다.

```python
# password_utils.py 하단에 추가

class PasswordPolicy:
    """
    비밀번호 정책을 정의하는 클래스입니다.

    Attributes:
        min_length: 최소 비밀번호 길이. 기본값 8.
        require_uppercase: 대문자 포함 여부. 기본값 True.
        require_digit: 숫자 포함 여부. 기본값 True.

    Example:
        >>> policy = PasswordPolicy(min_length=10, require_digit=False)
        >>> policy.validate("HelloWorld!")
        True
    """

    def __init__(
        self,
        min_length: int = 8,
        require_uppercase: bool = True,
        require_digit: bool = True,
    ) -> None:
        self.min_length = min_length
        self.require_uppercase = require_uppercase
        self.require_digit = require_digit

    def validate(self, password: str) -> bool:
        """
        비밀번호가 현재 정책을 만족하는지 검사합니다.

        Args:
            password: 검사할 비밀번호 문자열.

        Returns:
            정책을 모두 만족하면 True, 하나라도 위반하면 False.
        """
        if len(password) < self.min_length:
            return False
        if self.require_uppercase and not any(c.isupper() for c in password):
            return False
        if self.require_digit and not any(c.isdigit() for c in password):
            return False
        return True
```

완성된 파일 전체를 AI에게 보여주고 다음을 요청하세요:

```
이 모듈 전체를 검토하고 모듈 수준 docstring을 맨 위에 추가해줘.
함수와 클래스의 docstring 중 부정확하거나 누락된 부분도 지적해줘.
```

AI의 피드백을 바탕으로 직접 수정합니다.

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 해결 방법 |
|------|-------------------|-----------|
| docstring을 함수 정의 *위에* 씀 | 문서가 `__doc__`에 연결되지 않음 | `def` 줄 바로 다음 줄, 함수 본문 첫 번째 줄에 작성 |
| `Example` 출력값이 실제와 다름 | `doctest` 실패: `Expected: ... Got: ...` | `python3 -m doctest 파일명.py`로 실행 후 실제 출력값으로 수정 |
| 타입 힌트 없이 `Args` 타입 설명 누락 | 문서만 봐서는 타입을 알 수 없음 | 함수 시그니처에 `def f(x: int)` 형태로 타입 힌트 추가 |
| AI가 생성한 `Raises` 섹션이 실제 코드와 불일치 | 런타임에 다른 예외 발생 | 함수 본문을 직접 읽고 실제로 발생 가능한 예외만 남김 |
| 삼중 따옴표를 `'` 세 개로 시작했다가 `"` 세 개로 닫음 | `SyntaxError: EOL while scanning string literal` | 시작과 끝을 같은 따옴표로 통일: `"""..."""` 또는 `'''...'''` |
| 빈 줄 없이 `Args:` 섹션 바로 시작 | IDE 일부에서 요약 문장이 Args와 붙어서 표시됨 | 요약 한 줄 → 빈 줄 → `Args:` 순서로 작성 |
| 모든 함수에 자명한 docstring 추가 | `def get_name(): """이름을 가져옵니다."""` — 가치 없음 | 한 줄 요약이 함수 이름의 반복이면 생략하거나 더 구체적으로 작성 |

---

## 확인 체크리스트

- [ ] 내가 작성한 모든 `public` 함수에 docstring이 있다
- [ ] 각 docstring의 첫 줄은 마침표로 끝나는 완전한 한 문장이다
- [ ] `Args` 섹션에 모든 파라미터가 설명되어 있다
- [ ] `Returns` 섹션에 반환값의 형태와 의미가 적혀 있다
- [ ] 예외를 발생시키는 함수에는 `Raises` 섹션이 있다
- [ ] `Example` 섹션의 코드를 복사해서 실제로 실행해보았다
- [ ] `python3 -m doctest 파일명.py`를 실행했을 때 오류가 없다
- [ ] AI가 생성한 docstring을 그대로 쓰지 않고 직접 검토·수정했다
- [ ] `help(내함수이름)` 또는 IDE의 자동완성에서 문서가 올바르게 표시된다

---

## 한 번 더 생각해 보기

1. AI가 생성한 `Example` 섹션이 실제 실행 결과와 다를 수 있는 이유는 무엇일까요? 이 문제를 예방하려면 어떤 습관을 들이면 좋을까요?

2. 함수 이름이 이미 `calculate_monthly_tax_from_income`처럼 충분히 명확할 때도 docstring이 필요할까요? 어떤 기준으로 판단하면 좋을지 생각해 보세요.

3. 코드를 수정할 때 docstring도 함께 수정하지 않으면 어떤 문제가 생길까요? 팀 프로젝트에서 이를 방지하는 방법을 아는 대로 이야기해 보세요.

---

## 다음 장

다음 장에서는 `pytest`를 사용해 자동화된 테스트를 작성하고, 방금 문서화한 `Example` 섹션을 테스트 케이스로 활용하는 방법을 배웁니다.