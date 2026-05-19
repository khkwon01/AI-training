## 이 장에서 배우는 것

- `dataclass`를 사용해 데이터를 담는 클래스를 간결하게 만드는 방법
- `namedtuple`로 이름 있는 튜플을 정의하는 방법
- 두 방식의 차이점과 각각 언제 쓰면 좋은지
- 기본값, 타입 힌트, 불변(immutable) 설정 방법
- 실제 프로그램에서 데이터를 구조화하는 실전 패턴

---

## 먼저 쉬운 설명

프로그램을 만들다 보면 데이터를 묶어서 다뤄야 할 때가 많습니다. 예를 들어 학생 정보를 저장할 때 이름, 나이, 점수를 각각 따로 변수에 넣으면 코드가 금방 복잡해집니다.

```python
# 이렇게 쓰면 관리하기 힘들어요
student_name = "김민준"
student_age = 20
student_score = 95
```

이럴 때 데이터를 하나의 묶음으로 표현하는 도구가 있습니다. Python에는 **data class**와 **named tuple** 두 가지가 있는데, 마치 "데이터 전용 서랍장"을 만드는 방법입니다. 직접 클래스를 처음부터 작성하는 것보다 훨씬 적은 코드로 깔끔하게 만들 수 있습니다.

---

## 1. dataclass 기본 사용법

`@dataclass` 데코레이터를 붙이면 Python이 `__init__`, `__repr__` 같은 메서드를 자동으로 만들어 줍니다.

```python
# student.py
from dataclasses import dataclass

@dataclass
class Student:
    name: str
    age: int
    score: float

# 사용 예시
s = Student(name="김민준", age=20, score=95.5)
print(s)           # Student(name='김민준', age=20, score=95.5)
print(s.name)      # 김민준
print(s.score)     # 95.5
```

`@dataclass`가 없는 일반 클래스와 비교해 보면 차이가 확연합니다.

```python
# @dataclass 없이 같은 기능을 구현하면 이렇게 길어져요
class StudentOld:
    def __init__(self, name: str, age: int, score: float):
        self.name = name
        self.age = age
        self.score = score

    def __repr__(self):
        return f"StudentOld(name={self.name!r}, age={self.age!r}, score={self.score!r})"
```

---

## 2. 기본값과 타입 힌트 설정

필드에 기본값을 지정하면 인스턴스를 만들 때 해당 인수를 생략할 수 있습니다.

```python
# product.py
from dataclasses import dataclass, field

@dataclass
class Product:
    name: str
    price: float
    in_stock: bool = True          # 기본값 지정
    tags: list = field(default_factory=list)  # 가변 기본값은 field() 사용

p1 = Product(name="노트북", price=1200000)
p2 = Product(name="마우스", price=35000, in_stock=False, tags=["주변기기", "무선"])

print(p1)  # Product(name='노트북', price=1200000, in_stock=True, tags=[])
print(p2)  # Product(name='마우스', price=35000, in_stock=False, tags=['주변기기', '무선'])
```

> **주의:** 리스트나 딕셔너리처럼 변경 가능한(mutable) 타입을 기본값으로 쓸 때는 반드시 `field(default_factory=list)` 형태를 써야 합니다. 아래 `## 자주 하는 실수` 섹션에서 자세히 다룹니다.

---

## 3. frozen=True로 불변 데이터 만들기

`@dataclass(frozen=True)`로 설정하면 생성 후 값을 변경할 수 없는 불변 객체가 됩니다. 설정값이나 좌표처럼 바뀌면 안 되는 데이터에 적합합니다.

```python
# config.py
from dataclasses import dataclass

@dataclass(frozen=True)
class DatabaseConfig:
    host: str
    port: int
    database: str

config = DatabaseConfig(host="localhost", port=5432, database="mydb")
print(config.host)  # localhost

# 값을 바꾸려고 하면 에러 발생
config.port = 3306
# FrozenInstanceError: cannot assign to field 'port'
```

`frozen=True`인 dataclass는 딕셔너리 키나 세트 원소로도 사용할 수 있습니다 (해시 가능).

---

## 4. namedtuple 기본 사용법

`namedtuple`은 튜플에 필드 이름을 붙인 것입니다. 한 번 만들면 값을 바꿀 수 없고, 인덱스와 이름 모두로 접근할 수 있습니다.

```python
# point.py
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])

p = Point(x=3, y=7)
print(p)       # Point(x=3, y=7)
print(p.x)     # 3
print(p[0])    # 3  (튜플처럼 인덱스 접근도 가능)
print(p == (3, 7))  # True  (일반 튜플과 비교 가능)
```

Python 3.6 이상에서는 `typing.NamedTuple`을 써서 타입 힌트와 기본값을 더 명확하게 지정할 수 있습니다.

```python
# order.py
from typing import NamedTuple

class Order(NamedTuple):
    order_id: str
    item: str
    quantity: int
    discount: float = 0.0  # 기본값 지정 가능

o1 = Order(order_id="ORD-001", item="키보드", quantity=1)
o2 = Order(order_id="ORD-002", item="모니터", quantity=2, discount=0.1)

print(o1)  # Order(order_id='ORD-001', item='키보드', quantity=1, discount=0.0)
print(o2.discount)  # 0.1
```

---

## 5. dataclass vs namedtuple 비교

두 도구의 특성을 알면 상황에 맞게 선택할 수 있습니다.

```python
# comparison.py
from dataclasses import dataclass
from typing import NamedTuple

@dataclass
class MutablePoint:
    x: float
    y: float

class ImmutablePoint(NamedTuple):
    x: float
    y: float

mp = MutablePoint(1.0, 2.0)
mp.x = 99.0          # 변경 가능
print(mp.x)          # 99.0

ip = ImmutablePoint(1.0, 2.0)
ip.x = 99.0          # AttributeError: can't set attribute
```

| 항목 | `@dataclass` | `NamedTuple` |
|------|-------------|--------------|
| 값 변경 | 가능 (기본) | 불가능 |
| 메서드 추가 | 자유롭게 가능 | 가능하지만 제한적 |
| 튜플 언패킹 | 불가능 | 가능 |
| 딕셔너리 키 사용 | `frozen=True` 필요 | 기본 가능 |
| 메모리 효율 | 보통 | 약간 더 적음 |
| 추천 상황 | 복잡한 로직이 포함된 데이터 | 간단한 값 묶음, 반환값 |

---

## 6. 실전 예시 — 함수 반환값으로 활용하기

여러 값을 반환할 때 `NamedTuple`을 쓰면 코드 가독성이 크게 높아집니다.

```python
# grade_calculator.py
from typing import NamedTuple

class GradeResult(NamedTuple):
    average: float
    grade: str
    passed: bool

def calculate_grade(scores: list[int]) -> GradeResult:
    avg = sum(scores) / len(scores)
    if avg >= 90:
        grade = "A"
    elif avg >= 80:
        grade = "B"
    elif avg >= 70:
        grade = "C"
    else:
        grade = "F"
    return GradeResult(average=avg, grade=grade, passed=avg >= 60)

result = calculate_grade([85, 92, 78, 95])
print(result.average)  # 87.5
print(result.grade)    # B
print(result.passed)   # True

# 언패킹도 가능
avg, grade, passed = calculate_grade([45, 50, 55])
print(f"점수: {avg}, 등급: {grade}, 합격: {passed}")
```

---

## 따라 하기 실습

### 실습 1 — 도서 정보 dataclass 만들기

`book_manager.py` 파일을 만들고 아래 내용을 작성합니다.

```python
# book_manager.py
from dataclasses import dataclass, field

@dataclass
class Book:
    title: str
    author: str
    price: int
    genres: list = field(default_factory=list)

    def discounted_price(self, rate: float) -> int:
        """할인율(0.0 ~ 1.0)을 받아 할인된 가격을 반환합니다."""
        return int(self.price * (1 - rate))

# 책 목록 만들기
books = [
    Book(title="파이썬 완전 정복", author="홍길동", price=28000, genres=["프로그래밍"]),
    Book(title="데이터 분석 입문", author="이순신", price=32000, genres=["데이터", "통계"]),
    Book(title="알고리즘 기초", author="강감찬", price=25000),
]

for book in books:
    print(book)
    print(f"  10% 할인가: {book.discounted_price(0.1):,}원")
```

실행 결과를 확인한 뒤, `genres`가 없는 책의 `genres`가 `[]`로 출력되는지 확인하세요.

---

### 실습 2 — 위치 좌표 NamedTuple로 계산하기

`location_calc.py` 파일을 만들고 두 지점 사이의 거리를 계산합니다.

```python
# location_calc.py
import math
from typing import NamedTuple

class Location(NamedTuple):
    name: str
    latitude: float
    longitude: float

def distance_km(a: Location, b: Location) -> float:
    """두 위치 사이의 직선 거리(km)를 근사값으로 계산합니다."""
    lat_diff = (a.latitude - b.latitude) * 111.0
    lon_diff = (a.longitude - b.longitude) * 88.8
    return round(math.sqrt(lat_diff**2 + lon_diff**2), 2)

seoul = Location(name="서울", latitude=37.5665, longitude=126.9780)
busan = Location(name="부산", latitude=35.1796, longitude=129.0756)
daegu = Location(name="대구", latitude=35.8714, longitude=128.6014)

cities = [seoul, busan, daegu]
for city in cities:
    dist = distance_km(seoul, city)
    print(f"서울 → {city.name}: 약 {dist} km")
```

실습 1에서 만든 `Book`과 비교하면서 `NamedTuple`은 언패킹이 된다는 점도 확인해 보세요.

```python
name, lat, lon = seoul
print(name, lat, lon)  # 서울 37.5665 126.978
```

---

### 실습 3 — dataclass로 간단한 장바구니 구현하기

실습 1과 2를 바탕으로 `cart.py`를 작성합니다. `Cart` dataclass 안에 `Book` 리스트를 담고 총액을 계산하는 메서드를 추가합니다.

```python
# cart.py
from dataclasses import dataclass, field
from book_manager import Book  # 실습 1 파일 재사용

@dataclass
class Cart:
    owner: str
    items: list[Book] = field(default_factory=list)

    def add(self, book: Book) -> None:
        self.items.append(book)

    def total_price(self) -> int:
        return sum(book.price for book in self.items)

    def summary(self) -> str:
        lines = [f"[{self.owner}의 장바구니]"]
        for book in self.items:
            lines.append(f"  - {book.title}: {book.price:,}원")
        lines.append(f"  합계: {self.total_price():,}원")
        return "\n".join(lines)

cart = Cart(owner="김민준")
cart.add(Book(title="파이썬 완전 정복", author="홍길동", price=28000))
cart.add(Book(title="알고리즘 기초", author="강감찬", price=25000))
print(cart.summary())
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 | 원인 및 해결 방법 |
|------|------------|------------------|
| 기본값으로 빈 리스트 직접 사용 | `ValueError: mutable default <class 'list'> for field tags is not allowed` | 모든 인스턴스가 같은 리스트 객체를 공유하게 됨. `field(default_factory=list)` 로 교체 |
| 기본값 없는 필드를 기본값 있는 필드 뒤에 선언 | `TypeError: non-default argument 'name' follows default argument` | 기본값이 없는 필드는 항상 앞에 와야 함. 순서를 바꿔서 해결 |
| `frozen=True` 객체의 값 변경 시도 | `FrozenInstanceError: cannot assign to field 'port'` | `frozen=True`는 불변 설정. 값 변경이 필요하면 `frozen=True` 제거 또는 `dataclasses.replace()` 사용 |
| NamedTuple 필드 값 변경 시도 | `AttributeError: can't set attribute` | NamedTuple은 튜플이라 불변. 변경이 필요하면 `_replace()` 메서드 사용 |
| `@dataclass` 없이 타입 힌트만 작성 | `__init__` 이 자동 생성되지 않아 `TypeError: __init__() takes 1 positional argument` | 데코레이터 `@dataclass`를 클래스 위에 추가 |
| `from dataclasses import dataclass` 누락 | `NameError: name 'dataclass' is not defined` | 파일 상단에 임포트 구문 추가 |

---

## 확인 체크리스트

- [ ] `@dataclass` 데코레이터를 붙이면 `__init__`과 `__repr__`이 자동 생성된다는 것을 안다
- [ ] 리스트나 딕셔너리를 기본값으로 쓸 때 `field(default_factory=list)`를 사용할 수 있다
- [ ] `frozen=True` 옵션으로 불변 dataclass를 만들 수 있다
- [ ] `NamedTuple`을 정의하고 이름과 인덱스 모두로 필드에 접근할 수 있다
- [ ] dataclass는 값 변경이 가능하고, NamedTuple은 불변이라는 차이를 설명할 수 있다
- [ ] 함수에서 여러 값을 반환할 때 NamedTuple을 활용할 수 있다
- [ ] `field(default_factory=...)` 없이 리스트 기본값을 쓰면 왜 문제가 되는지 설명할 수 있다
- [ ] 실습 3의 `Cart` 클래스를 직접 처음부터 작성할 수 있다

---

## 한 번 더 생각해 보기

1. `frozen=True`인 dataclass와 `NamedTuple` 모두 불변입니다. 그렇다면 둘 중 어느 것을 선택할 때 고민이 생길까요? 각각 어떤 상황에서 더 자연스럽게 어울릴지 예를 들어 생각해 보세요.

2. `Cart` 클래스에 `items: list[Book]`처럼 다른 dataclass를 필드로 넣었습니다. 만약 `Book` 클래스를 `frozen=True`로 바꾸면 `Cart` 동작에 어떤 영향이 있을까요? 직접 코드를 수정해서 확인해 보세요.

3. 일반 딕셔너리(`dict`)와 `NamedTuple`을 비교해 보세요. API 응답 데이터를 프로그램 안에서 다룰 때 딕셔너리 대신 `NamedTuple`이나 `dataclass`를 쓰면 어떤 장점이 생길까요?

---

## 다음 장

다음 장에서는 Python의 `enum` 모듈로 고정된 상수 집합을 안전하게 표현하는 방법을 배웁니다.