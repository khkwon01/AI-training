## 이 장에서 배우는 것

- `pydantic`이 무엇인지, 왜 쓰는지 이해한다
- `BaseModel`을 상속해 데이터 모델 클래스를 만든다
- 타입 어노테이션으로 자동 유효성 검사를 수행한다
- `ValidationError`가 발생했을 때 어디서 문제가 생겼는지 파악한다
- `Optional`, 기본값, `Field`를 활용해 유연한 모델을 작성한다
- 중첩 모델(모델 안의 모델)을 만들어 복잡한 데이터를 다룬다

---

## 먼저 쉬운 설명

회원가입 폼을 만든다고 생각해 보세요. 사용자가 나이 칸에 `"스물다섯"`이라고 입력하면 어떻게 될까요? 숫자가 와야 할 자리에 문자가 들어와서 프로그램이 엉망이 됩니다.

지금까지는 이런 상황을 막으려면 이렇게 직접 검사해야 했습니다.

```python
def create_user(name, age, email):
    if not isinstance(name, str):
        raise ValueError("name은 문자열이어야 합니다")
    if not isinstance(age, int):
        raise ValueError("age는 정수여야 합니다")
    if age < 0:
        raise ValueError("age는 0 이상이어야 합니다")
    # ... 이런 코드가 끝없이 이어진다
```

필드가 10개, 20개가 되면 이런 코드를 유지하는 것은 끔찍합니다.

`pydantic`은 이 반복 작업을 **타입 어노테이션 한 줄**로 대체해 줍니다. 잘못된 데이터가 들어오면 자동으로 잡아내고, 어디서 무엇이 잘못됐는지 친절하게 알려 줍니다. FastAPI 같은 현대적인 웹 프레임워크가 `pydantic`을 기본으로 채택한 이유가 바로 이것입니다.

---

## 1. pydantic 설치와 첫 번째 모델

먼저 `pydantic`을 설치합니다.

```bash
pip install pydantic
```

이제 가장 간단한 모델을 만들어 봅시다.

```python
# user_model.py
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
    email: str

# 올바른 데이터로 인스턴스 생성
user = User(name="김민준", age=25, email="minjun@example.com")
print(user)
# name='김민준' age=25 email='minjun@example.com'

print(user.name)   # 김민준
print(user.age)    # 25
```

`BaseModel`을 상속받고, 클래스 변수에 타입 어노테이션을 붙이는 것이 전부입니다. `pydantic`이 나머지를 알아서 처리합니다.

---

## 2. 잘못된 데이터가 들어오면 어떻게 되나

`pydantic`의 진짜 힘은 **잘못된 데이터를 거절할 때** 나타납니다.

```python
# validation_error_demo.py
from pydantic import BaseModel, ValidationError

class User(BaseModel):
    name: str
    age: int
    email: str

# 잘못된 데이터를 넣어 보자
try:
    bad_user = User(name="이수아", age="스물다섯", email="sua@example.com")
except ValidationError as e:
    print(e)
```

실행하면 다음과 같은 오류 메시지가 출력됩니다.

```
1 validation error for User
age
  Input should be a valid integer, unable to parse string as an integer
  [type=int_parsing, input_value='스물다섯', input_url=...]
```

어떤 필드(`age`)에서 어떤 값(`'스물다섯'`)이 왜 문제인지(`unable to parse string as an integer`) 한 눈에 보입니다. 여러 필드가 동시에 틀렸다면 모든 오류를 한꺼번에 알려 줍니다.

```python
# 여러 필드가 동시에 잘못된 경우
try:
    bad_user = User(name=123, age="틀림", email=None)
except ValidationError as e:
    print(f"오류 개수: {e.error_count()}")
    for error in e.errors():
        print(f"  필드: {error['loc']}, 메시지: {error['msg']}")
```

---

## 3. Optional 필드와 기본값

모든 필드가 필수일 필요는 없습니다. 선택 사항인 필드는 `Optional`과 기본값을 함께 씁니다.

```python
# optional_fields.py
from pydantic import BaseModel
from typing import Optional

class UserProfile(BaseModel):
    name: str
    age: int
    email: str
    bio: Optional[str] = None          # 없어도 되는 필드, 기본값 None
    is_active: bool = True             # 기본값이 있는 필드
    login_count: int = 0

# bio 없이 생성해도 오류 없음
user = UserProfile(name="박지호", age=30, email="jiho@example.com")
print(user.bio)          # None
print(user.is_active)    # True
print(user.login_count)  # 0

# bio를 직접 지정할 수도 있음
user2 = UserProfile(
    name="최유나",
    age=22,
    email="yuna@example.com",
    bio="안녕하세요, 파이썬 개발자입니다."
)
print(user2.bio)  # 안녕하세요, 파이썬 개발자입니다.
```

---

## 4. Field로 세밀한 규칙 추가하기

타입 검사만으로는 부족할 때가 있습니다. 나이가 `int`이지만 음수면 안 되고, 이름이 `str`이지만 빈 문자열이면 곤란합니다. `Field`를 사용하면 이런 세밀한 규칙을 추가할 수 있습니다.

```python
# field_validation.py
from pydantic import BaseModel, Field
from typing import Optional

class Product(BaseModel):
    name: str = Field(min_length=1, max_length=100, description="상품 이름")
    price: float = Field(gt=0, description="가격은 0보다 커야 합니다")
    stock: int = Field(ge=0, description="재고는 0 이상이어야 합니다")
    discount_rate: float = Field(default=0.0, ge=0.0, le=1.0, description="할인율 0~1")

# 올바른 상품
item = Product(name="무선 키보드", price=89000, stock=50, discount_rate=0.1)
print(item)

# 가격에 음수를 넣으면?
try:
    bad_item = Product(name="이상한 상품", price=-1000, stock=10)
except Exception as e:
    print(e)
# price
#   Input should be greater than 0 [type=greater_than, ...]
```

자주 쓰는 `Field` 옵션을 정리하면 다음과 같습니다.

| 옵션 | 의미 | 예시 |
|------|------|------|
| `gt` | 초과 (greater than) | `gt=0` → 0보다 큰 값 |
| `ge` | 이상 (greater or equal) | `ge=0` → 0 이상 |
| `lt` | 미만 (less than) | `lt=100` → 100 미만 |
| `le` | 이하 (less or equal) | `le=1.0` → 1.0 이하 |
| `min_length` | 문자열 최소 길이 | `min_length=1` |
| `max_length` | 문자열 최대 길이 | `max_length=50` |

---

## 5. 중첩 모델: 모델 안에 모델

실제 데이터는 단순하지 않습니다. 주문 정보에는 고객 정보와 배송지 정보가 함께 들어 있는 것처럼, 모델 안에 다른 모델을 넣을 수 있습니다.

```python
# nested_model.py
from pydantic import BaseModel, Field
from typing import List

class Address(BaseModel):
    city: str
    zipcode: str = Field(pattern=r"^\d{5}$", description="우편번호 5자리 숫자")
    street: str

class OrderItem(BaseModel):
    product_name: str
    quantity: int = Field(gt=0)
    unit_price: float = Field(gt=0)

class Order(BaseModel):
    order_id: str
    customer_name: str
    shipping_address: Address        # Address 모델을 중첩
    items: List[OrderItem]           # OrderItem 리스트

# 딕셔너리로도 생성 가능 (pydantic이 자동 변환)
order_data = {
    "order_id": "ORD-2024-001",
    "customer_name": "홍길동",
    "shipping_address": {
        "city": "서울",
        "zipcode": "04524",
        "street": "강남대로 123"
    },
    "items": [
        {"product_name": "노트북", "quantity": 1, "unit_price": 1500000},
        {"product_name": "마우스", "quantity": 2, "unit_price": 35000},
    ]
}

order = Order(**order_data)
print(order.customer_name)                   # 홍길동
print(order.shipping_address.city)           # 서울
print(order.items[0].product_name)           # 노트북
print(order.model_dump())                    # 전체를 딕셔너리로 변환
```

---

## 따라 하기 실습

### 실습 1 — 회원 가입 모델 만들기

`signup_model.py` 파일을 만들고 아래 요구 사항을 구현하세요.

**요구 사항:**
- `username`: 문자열, 3자 이상 20자 이하
- `password`: 문자열, 8자 이상
- `age`: 정수, 14 이상 (만 14세 이상만 가입 가능)
- `email`: 문자열 (필수)
- `newsletter`: 불리언, 기본값 `False`

```python
# signup_model.py
from pydantic import BaseModel, Field

class SignUpForm(BaseModel):
    username: str = Field(min_length=3, max_length=20)
    password: str = Field(min_length=8)
    age: int = Field(ge=14)
    email: str
    newsletter: bool = False

# 아래 코드로 테스트해 보세요
valid = SignUpForm(
    username="minjun_k",
    password="securePass1",
    age=25,
    email="minjun@example.com"
)
print("가입 성공:", valid.username)

# 나이가 10살인 경우
from pydantic import ValidationError
try:
    invalid = SignUpForm(
        username="child",
        password="pass1234",
        age=10,
        email="child@example.com"
    )
except ValidationError as e:
    print("가입 실패:", e)
```

---

### 실습 2 — 상품 목록 검증기 만들기

실습 1을 바탕으로 `product_validator.py`를 만드세요. 여러 상품을 담은 리스트를 받아 유효한 것과 유효하지 않은 것을 분류합니다.

```python
# product_validator.py
from pydantic import BaseModel, Field, ValidationError
from typing import List, Optional

class Product(BaseModel):
    name: str = Field(min_length=1, max_length=100)
    price: float = Field(gt=0)
    stock: int = Field(ge=0)
    category: Optional[str] = None

def validate_products(raw_list: List[dict]):
    valid_products = []
    invalid_products = []

    for i, data in enumerate(raw_list):
        try:
            product = Product(**data)
            valid_products.append(product)
        except ValidationError as e:
            invalid_products.append({"index": i, "data": data, "errors": e.errors()})

    return valid_products, invalid_products

# 테스트 데이터 (일부러 잘못된 항목 포함)
sample_data = [
    {"name": "무선 마우스", "price": 35000, "stock": 100},
    {"name": "", "price": 15000, "stock": 50},          # 이름이 빈 문자열
    {"name": "키보드", "price": -5000, "stock": 30},    # 가격이 음수
    {"name": "모니터", "price": 450000, "stock": 0, "category": "디스플레이"},
]

valid, invalid = validate_products(sample_data)
print(f"유효한 상품: {len(valid)}개")
print(f"유효하지 않은 상품: {len(invalid)}개")
for item in invalid:
    print(f"  [{item['index']}번] {item['data']['name']!r} — {item['errors'][0]['msg']}")
```

---

### 실습 3 — 주문 처리 시스템 완성하기

실습 1과 2를 합쳐 `order_system.py`를 만드세요. 회원이 주문을 넣으면 회원 정보와 주문 내역을 함께 검증합니다.

```python
# order_system.py
from pydantic import BaseModel, Field
from typing import List
from datetime import datetime

class CustomerInfo(BaseModel):
    name: str = Field(min_length=1)
    email: str
    phone: str = Field(pattern=r"^010-\d{4}-\d{4}$")  # 010-XXXX-XXXX 형식

class CartItem(BaseModel):
    product_id: str
    name: str
    quantity: int = Field(gt=0)
    price: float = Field(gt=0)

    def subtotal(self) -> float:
        return self.quantity * self.price

class OrderRequest(BaseModel):
    customer: CustomerInfo
    items: List[CartItem] = Field(min_length=1)  # 최소 1개 이상

    def total_price(self) -> float:
        return sum(item.subtotal() for item in self.items)

# 실제 주문 요청 테스트
order_request = OrderRequest(
    customer={
        "name": "이수진",
        "email": "sujin@example.com",
        "phone": "010-1234-5678"
    },
    items=[
        {"product_id": "P001", "name": "에어팟", "quantity": 1, "price": 199000},
        {"product_id": "P002", "name": "케이스", "quantity": 2, "price": 15000},
    ]
)

print(f"주문자: {order_request.customer.name}")
print(f"총 금액: {order_request.total_price():,.0f}원")

# 잘못된 전화번호 형식 테스트
from pydantic import ValidationError
try:
    bad_order = OrderRequest(
        customer={"name": "홍길동", "email": "hong@example.com", "phone": "01012345678"},
        items=[{"product_id": "P001", "name": "상품", "quantity": 1, "price": 1000}]
    )
except ValidationError as e:
    print(f"\n오류: {e.errors()[0]['msg']}")
```

---

## 자주 하는 실수

| 실수 | 실제 오류 메시지 | 해결 방법 |
|------|-----------------|-----------|
| `BaseModel`을 import하지 않음 | `NameError: name 'BaseModel' is not defined` | `from pydantic import BaseModel` 추가 |
| 타입 어노테이션 없이 필드 선언 | 필드로 인식되지 않고 클래스 변수로 처리됨 | `name: str`처럼 반드시 타입 명시 |
| 필수 필드를 빠뜨리고 생성 | `ValidationError: Field required [type=missing]` | 필수 필드를 모두 전달하거나 `Optional`과 기본값 설정 |
| `Optional` 없이 `None` 기본값 설정 | pydantic v2에서 경고 또는 오류 발생 | `Optional[str] = None` 형태로 작성 |
| `Field(gt=0)`인데 0 전달 | `Input should be greater than 0 [type=greater_than]` | `ge=0`(이상)과 `gt=0`(초과) 구분 |
| 딕셔너리를 직접 비교 | `user == {"name": "김민준"}` → `False` | `user.model_dump() == {...}` 사용 |
| pydantic v1 문법을 v2에서 사용 | `PydanticUserError: ... moved to model_config` | `.dict()` → `.model_dump()`, `.schema()` → `.model_json_schema()` |

---

## 확인 체크리스트

- [ ] `from pydantic import BaseModel`로 `BaseModel`을 가져올 수 있다
- [ ] `BaseModel`을 상속한 클래스에 타입 어노테이션으로 필드를 선언할 수 있다
- [ ] 잘못된 타입의 데이터를 넣으면 `ValidationError`가 발생함을 안다
- [ ] `try / except ValidationError`로 오류를 잡아 처리할 수 있다
- [ ] `Optional[타입] = None`으로 선택 필드를 만들 수 있다
- [ ] `Field`로 `gt`, `ge`, `min_length` 같은 추가 제약 조건을 설정할 수 있다
- [ ] 모델 안에 다른 모델을 필드로 중첩할 수 있다
- [ ] `.model_dump()`로 모델 인스턴스를 딕셔너리로 변환할 수 있다
- [ ] 실습 3의 `order_system.py`를 오류 없이 실행했다

---

## 한 번 더 생각해 보기

1. `pydantic`이 없던 시절에는 데이터 유효성 검사를 어떻게 했을까요? 직접 `if isinstance(...)` 코드를 짜는 것과 `pydantic`을 쓰는 것의 차이를 팀원에게 설명해 보세요.

2. 실습 2에서 유효하지 않은 상품 데이터를 만났을 때 프로그램이 즉시 중단되지 않고 계속 실행된 이유는 무엇인가요? `try / except` 블록이 없었다면 어떻게 됐을지 생각해 보세요.

3. 중첩 모델(`Address` 안에 `City` 모델, `Order` 안에 `Address` 모델)을 여러 단계로 쌓으면 어떤 장점이 있을까요? 반면 모든 필드를 하나의 평평한(flat) 모델에 넣으면 어떤 문제가 생길까요?

---

## 다음 장

다음 장에서는 `pydantic`으로 검증한 데이터를 **FastAPI 엔드포인트**에 연결해, 실제 HTTP 요청을 받아 처리하는 웹 API를 만드는 방법을 배웁니다.