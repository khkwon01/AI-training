## 이 장에서 배우는 것

- 아주 작은 Python 함수 하나가 어떻게 하나의 서비스 기능이 될 수 있는지 이해한다
- 함수를 읽고, 무슨 일을 하는지 스스로 설명할 수 있다
- 함수를 직접 실행해 보고 결과를 확인한다
- 이 작은 코드가 나중에 어떻게 더 커질 수 있는지 상상해 본다

---

## 먼저 쉬운 설명

"서비스 코드"라고 하면 왠지 거대하고 복잡한 것처럼 느껴질 수 있어요.

하지만 실제 서비스는 아주 작은 기능들이 모여서 만들어집니다. 예를 들어 쇼핑몰 앱에도 이런 작은 기능들이 있어요:

- 할인 가격을 계산한다
- 이름이 비어 있는지 확인한다
- 주문 번호를 만들어 준다

이 중에서 하나만 골라서 코드로 써도, 그게 이미 "서비스 기능"입니다.

이 장에서는 **"할인 가격 계산"** 이라는 딱 하나의 기능을 코드로 어떻게 표현하는지 살펴봅니다. 짧고 간단하지만, 실제 서비스에서도 이렇게 시작합니다.

---

## 1. 가장 작은 서비스 기능 — 할인 가격 계산기

### 코드를 보기 전에

우리가 만들 기능을 말로 먼저 써봅시다:

> "원래 가격과 할인율을 받아서, 할인된 가격을 돌려준다."

이걸 Python으로 옮기면 다음과 같습니다.

```python
# discount.py

def calculate_discounted_price(price: float, discount_rate: float) -> float:
    """
    원래 가격에서 할인율을 적용한 최종 가격을 반환한다.

    price        : 원래 가격 (예: 10000)
    discount_rate: 할인율, 0~1 사이 숫자 (예: 0.1 = 10% 할인)
    반환값       : 할인 적용 후 가격
    """
    discounted = price * (1 - discount_rate)
    return discounted


# 직접 실행해서 확인해 보기
if __name__ == "__main__":
    result = calculate_discounted_price(10000, 0.1)
    print(f"할인 후 가격: {result}원")  # 출력: 할인 후 가격: 9000.0원
```

### 코드를 한 줄씩 읽어보기

| 줄 | 하는 일 |
|---|---|
| `def calculate_discounted_price(...)` | 함수를 선언한다. 이름만 봐도 무엇을 하는지 알 수 있다 |
| `price: float` | 가격을 받는다. `float`는 소수점이 있는 숫자 형식이다 |
| `discount_rate: float` | 할인율을 받는다. 0.1이면 10% 할인이다 |
| `-> float` | 이 함수가 숫자를 돌려준다는 의미다 |
| `discounted = price * (1 - discount_rate)` | 실제 계산이다. 10000 × (1 - 0.1) = 9000 |
| `return discounted` | 계산 결과를 밖으로 보내준다 |

---

## 2. 입력값이 잘못됐을 때를 생각하기

지금 코드는 정상적인 값이 들어올 때만 제대로 동작합니다. 만약 할인율이 `-0.5`나 `2.0`처럼 이상한 값이 들어오면 어떻게 될까요?

서비스 코드는 이런 경우도 대비해야 합니다.

```python
# discount.py (개선 버전)

def calculate_discounted_price(price: float, discount_rate: float) -> float:
    """
    원래 가격에서 할인율을 적용한 최종 가격을 반환한다.
    discount_rate는 0 이상 1 이하여야 한다.
    """
    if price < 0:
        raise ValueError("가격은 0보다 작을 수 없습니다.")
    if not (0 <= discount_rate <= 1):
        raise ValueError("할인율은 0에서 1 사이여야 합니다.")

    discounted = price * (1 - discount_rate)
    return discounted


if __name__ == "__main__":
    # 정상 케이스
    print(calculate_discounted_price(10000, 0.1))   # 9000.0

    # 이상한 값 넣어보기
    print(calculate_discounted_price(10000, 1.5))   # ValueError 발생!
```

### 실행하면 이렇게 보입니다

```
9000.0
Traceback (most recent call last):
  File "discount.py", line 18, in <module>
    print(calculate_discounted_price(10000, 1.5))
  File "discount.py", line 9, in calculate_discounted_price
    raise ValueError("할인율은 0에서 1 사이여야 합니다.")
ValueError: 할인율은 0에서 1 사이여야 합니다.
```

에러가 발생하는 게 나쁜 것이 아닙니다. **잘못된 값이 들어왔을 때 명확하게 알려주는 것**이 좋은 서비스 코드의 특징입니다.

---

## 3. 이 기능을 다른 곳에서 가져다 쓰기

실제 서비스에서는 함수를 따로 파일에 저장하고, 필요한 곳에서 `import`해서 씁니다.

```python
# cart.py  ← 장바구니 기능을 담당하는 파일

from discount import calculate_discounted_price  # 방금 만든 함수를 가져온다

def show_cart_summary(items: list) -> None:
    """
    장바구니 목록을 출력한다.
    items: [{"name": "상품명", "price": 가격, "discount": 할인율}, ...]
    """
    print("=== 장바구니 ===")
    total = 0
    for item in items:
        final_price = calculate_discounted_price(item["price"], item["discount"])
        print(f"  {item['name']}: {item['price']}원 → {final_price}원")
        total += final_price
    print(f"합계: {total}원")


if __name__ == "__main__":
    my_cart = [
        {"name": "노트북 파우치", "price": 15000, "discount": 0.1},
        {"name": "USB 허브",     "price": 25000, "discount": 0.2},
    ]
    show_cart_summary(my_cart)
```

### 실행 결과

```
=== 장바구니 ===
  노트북 파우치: 15000원 → 13500.0원
  USB 허브: 25000원 → 20000.0원
합계: 33500.0원
```

`calculate_discounted_price` 함수 하나가 이제 장바구니 전체에서 재사용되고 있습니다. 이것이 작은 기능을 잘 만들어두는 이유입니다.

---

## 따라 하기 실습

### 실습 1 — 함수 직접 실행해 보기

1. 새 폴더를 만들고 `discount.py` 파일을 만든다
2. 아래 코드를 그대로 붙여넣는다

```python
# discount.py

def calculate_discounted_price(price: float, discount_rate: float) -> float:
    if price < 0:
        raise ValueError("가격은 0보다 작을 수 없습니다.")
    if not (0 <= discount_rate <= 1):
        raise ValueError("할인율은 0에서 1 사이여야 합니다.")
    return price * (1 - discount_rate)

if __name__ == "__main__":
    print(calculate_discounted_price(20000, 0.25))
```

3. 터미널에서 실행한다

```bash
python discount.py
```

기대 출력: `15000.0`

---

### 실습 2 — 값을 바꿔서 실험해 보기

`discount.py`의 `__main__` 블록을 아래처럼 바꿔 본다:

```python
if __name__ == "__main__":
    # 실험 1: 할인율 0% (아무 할인 없음)
    print(calculate_discounted_price(10000, 0.0))

    # 실험 2: 할인율 100% (공짜)
    print(calculate_discounted_price(10000, 1.0))

    # 실험 3: 잘못된 할인율 — 에러가 나는지 확인
    print(calculate_discounted_price(10000, -0.1))
```

각 실행 결과를 노트에 적어두고 왜 그 결과가 나왔는지 한 문장씩 설명해 본다.

---

### 실습 3 — 장바구니 파일 만들기

같은 폴더에 `cart.py`를 새로 만들고, 섹션 3의 코드를 붙여넣는다.

```bash
python cart.py
```

출력이 예시와 같은지 확인한다. 그 다음, 장바구니에 상품을 하나 더 추가하고 합계가 올바르게 바뀌는지 확인한다.

---

## 자주 하는 실수

| 실수 | 에러 메시지 | 해결 방법 |
|---|---|---|
| 할인율을 퍼센트 숫자로 넣음 | `ValueError: 할인율은 0에서 1 사이여야 합니다.` | `10`이 아니라 `0.1`로 입력한다 |
| `discount.py`가 없는데 `import` 함 | `ModuleNotFoundError: No module named 'discount'` | 두 파일이 같은 폴더에 있는지 확인한다 |
| 함수 이름 오타 | `NameError: name 'calculate_discoutned_price' is not defined` | 함수 이름 철자를 정확히 확인한다 |
| `return`을 빠뜨림 | 함수가 `None`을 반환해 합계가 이상해짐 | 계산 후 반드시 `return discounted`를 쓴다 |
| 들여쓰기가 맞지 않음 | `IndentationError: unexpected indent` | Python은 들여쓰기에 매우 민감하다. 스페이스 4칸을 일관되게 사용한다 |

---

## 확인 체크리스트

- [ ] `calculate_discounted_price(10000, 0.1)`을 실행하면 `9000.0`이 나온다
- [ ] 할인율에 `1.5`를 넣으면 `ValueError`가 발생한다
- [ ] `cart.py`에서 `discount.py`의 함수를 `import`해서 사용할 수 있다
- [ ] 함수가 하나의 기능만 담당하고 있다는 것을 설명할 수 있다
- [ ] `if __name__ == "__main__":` 블록이 왜 있는지 말할 수 있다
- [ ] 장바구니에 상품을 추가했을 때 합계가 올바르게 계산된다

---

## 한 번 더 생각해 보기

1. `calculate_discounted_price` 함수는 딱 한 가지 일만 합니다. 만약 이 함수가 "가격 계산"과 "결과를 화면에 출력"을 동시에 한다면 어떤 문제가 생길까요?

2. 지금은 할인율이 0~1 사이인지 확인합니다. 만약 가격이 소수점 이하로 너무 작은 값(예: `0.001원`)이 들어와도 괜찮을까요? 어떤 조건을 더 추가할 수 있을까요?

3. `cart.py`에서 상품이 100개라면 지금 코드로도 동작할까요? 코드를 바꾸지 않아도 된다면, 그 이유는 무엇일까요?

---

## 다음 장

다음 장에서는 이 작은 함수를 **여러 개 모아서 하나의 모듈로 구성하는 방법**을 배우고, 실제 프로젝트 폴더 구조가 어떻게 만들어지는지 살펴봅니다.