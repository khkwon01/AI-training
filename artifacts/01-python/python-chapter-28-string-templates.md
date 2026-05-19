## 이 장에서 배우는 것

- 파이썬에서 문자열을 만드는 세 가지 방법 (`%`, `.format()`, f-string)
- f-string을 실무에서 자유롭게 쓰는 방법
- 숫자, 날짜, 소수점 포맷 지정하기
- 여러 줄 문자열 템플릿 만들기
- `string.Template`으로 안전한 템플릿 다루기

---

## 먼저 쉬운 설명

프로그램을 만들다 보면 "안녕하세요, 김철수님! 잔액은 15,000원입니다." 같은 문장을 동적으로 만들어야 할 때가 정말 많습니다.

변수에 값이 들어있고, 그 값을 문장 중간에 끼워 넣어야 하죠.

예전에는 이렇게 했습니다.

```python
"안녕하세요, " + name + "님! 잔액은 " + str(balance) + "원입니다."
```

보기만 해도 복잡하죠? 파이썬은 이 문제를 해결하는 더 좋은 방법을 여러 가지 제공합니다. 이 장에서는 그 방법들을 처음부터 차근차근 배웁니다.

---

## 1. % 포맷 — 가장 오래된 방식

파이썬 초창기부터 있던 방식입니다. 지금도 오래된 코드에서 자주 보이기 때문에 읽을 수 있어야 합니다.

```python
# greet_old.py
name = "김철수"
age = 28
balance = 15000

message = "안녕하세요, %s님! 나이는 %d세이고, 잔액은 %d원입니다." % (name, age, balance)
print(message)
# 출력: 안녕하세요, 김철수님! 나이는 28세이고, 잔액은 15000원입니다.
```

| 기호 | 의미 |
|------|------|
| `%s` | 문자열 (string) |
| `%d` | 정수 (integer) |
| `%f` | 실수 (float) |
| `%.2f` | 소수점 둘째 자리까지 |

```python
# price_old.py
price = 3.14159
print("가격: %.2f원" % price)
# 출력: 가격: 3.14원
```

> **주의:** `%` 방식은 변수 순서가 조금만 틀려도 버그가 생기기 쉽습니다. 새 코드에서는 쓰지 않는 것을 권장합니다.

---

## 2. .format() — 조금 더 안전한 방식

파이썬 2.6부터 추가된 방법입니다. `{}` 자리표시자를 사용합니다.

```python
# greet_format.py
name = "이영희"
score = 95

# 위치로 지정
message = "{}님의 점수는 {}점입니다.".format(name, score)
print(message)
# 출력: 이영희님의 점수는 95점입니다.

# 이름으로 지정 (더 안전)
message2 = "{name}님의 점수는 {score}점입니다.".format(name=name, score=score)
print(message2)
# 출력: 이영희님의 점수는 95점입니다.
```

숫자 포맷도 지정할 수 있습니다.

```python
# number_format.py
salary = 3500000
rate = 0.035

print("월급: {:,}원".format(salary))        # 출력: 월급: 3,500,000원
print("이율: {:.1%}".format(rate))           # 출력: 이율: 3.5%
print("코드: {:05d}".format(42))             # 출력: 코드: 00042
```

---

## 3. f-string — 현재 가장 많이 쓰는 방식

파이썬 3.6부터 추가된 방법입니다. 문자열 앞에 `f`를 붙이고, `{}` 안에 변수나 식을 직접 씁니다.

```python
# greet_fstring.py
name = "박민준"
age = 32
city = "서울"

message = f"안녕하세요! 저는 {city}에 사는 {age}살 {name}입니다."
print(message)
# 출력: 안녕하세요! 저는 서울에 사는 32살 박민준입니다.
```

`{}` 안에서 계산도 바로 됩니다.

```python
# calc_fstring.py
price = 10000
quantity = 3
discount_rate = 0.1

total = price * quantity
discounted = total * (1 - discount_rate)

print(f"단가: {price:,}원")
print(f"수량: {quantity}개")
print(f"합계: {total:,}원")
print(f"할인 후: {discounted:,.0f}원")
print(f"할인율: {discount_rate:.0%}")
```

출력:
```
단가: 10,000원
수량: 3개
합계: 30,000원
할인 후: 27,000원
할인율: 10%
```

---

## 4. f-string 고급 포맷 지정

f-string의 `{}` 안에 `:` 뒤에 포맷 지시자를 씁니다.

```python
# format_spec.py
pi = 3.141592653589793
temperature = -5.678
user_id = 42

# 소수점 자리 지정
print(f"원주율: {pi:.4f}")           # 원주율: 3.1416
print(f"기온: {temperature:.1f}°C")  # 기온: -5.7°C

# 숫자 너비와 정렬
print(f"사용자 번호: {user_id:>10}")  # 오른쪽 정렬, 너비 10
print(f"사용자 번호: {user_id:<10}")  # 왼쪽 정렬, 너비 10
print(f"사용자 번호: {user_id:^10}")  # 가운데 정렬, 너비 10

# 0 채우기
print(f"주문번호: {user_id:08d}")     # 주문번호: 00000042
```

```python
# report_table.py
products = [
    ("사과", 1500, 10),
    ("바나나", 800, 25),
    ("포도", 3200, 5),
]

print(f"{'상품명':<8} {'단가':>8} {'수량':>6} {'소계':>10}")
print("-" * 36)
for name, price, qty in products:
    subtotal = price * qty
    print(f"{name:<8} {price:>7,}원 {qty:>5}개 {subtotal:>9,}원")
```

출력:
```
상품명        단가   수량         소계
------------------------------------
사과       1,500원    10개    15,000원
바나나       800원    25개    20,000원
포도       3,200원     5개    16,000원
```

---

## 5. 여러 줄 문자열 템플릿

이메일, 알림 메시지, SQL 같은 긴 문자열은 여러 줄로 작성합니다.

```python
# email_template.py
def make_welcome_email(name: str, team: str, start_date: str) -> str:
    return f"""
안녕하세요, {name}님!

{team} 팀에 오신 것을 진심으로 환영합니다.
입사일: {start_date}

첫 출근 전에 아래 사항을 확인해 주세요:
  1. 사원증 발급 (1층 총무팀)
  2. 노트북 수령 (2층 IT팀)
  3. 팀장님께 인사

궁금한 점이 있으면 언제든 연락 주세요.

감사합니다.
HR팀 드림
""".strip()

email = make_welcome_email("정수빈", "개발 1팀", "2026년 6월 2일")
print(email)
```

---

## 6. string.Template — 외부 입력에 안전한 방식

사용자 입력이나 설정 파일에서 온 문자열로 템플릿을 만들 때는 `string.Template`이 더 안전합니다. f-string은 코드 실행 중에 `{}` 안의 내용을 실행하기 때문에, 외부 입력을 그대로 쓰면 위험할 수 있습니다.

```python
# safe_template.py
from string import Template

# $변수명 또는 ${변수명} 형태를 사용
template = Template("안녕하세요, $name님! 오늘의 메뉴는 $menu입니다.")

# substitute: 변수가 빠지면 KeyError 발생
result = template.substitute(name="홍길동", menu="된장찌개")
print(result)
# 출력: 안녕하세요, 홍길동님! 오늘의 메뉴는 된장찌개입니다.

# safe_substitute: 변수가 없으면 그냥 원래 $변수명을 남겨둠
result2 = template.safe_substitute(name="홍길동")
print(result2)
# 출력: 안녕하세요, 홍길동님! 오늘의 메뉴는 $menu입니다.
```

```python
# config_template.py
from string import Template

# 설정 파일이나 DB에서 읽어온 템플릿이라고 가정
raw_template = "[$level] $timestamp - $message"

log_template = Template(raw_template)
log_line = log_template.substitute(
    level="INFO",
    timestamp="2026-05-18 09:00:00",
    message="서버가 시작되었습니다."
)
print(log_line)
# 출력: [INFO] 2026-05-18 09:00:00 - 서버가 시작되었습니다.
```

---

## 따라 하기 실습

### 실습 1 — 영수증 만들기

`receipt.py` 파일을 만들고 아래 코드를 작성하세요.

```python
# receipt.py
store_name = "파이썬 편의점"
items = [
    ("삼각김밥", 1200, 2),
    ("아메리카노", 2500, 1),
    ("물", 800, 3),
]

print(f"{'=' * 30}")
print(f"{'영수증':^30}")
print(f"매장명: {store_name}")
print(f"{'=' * 30}")
print(f"{'상품명':<10} {'단가':>6} {'수량':>4} {'금액':>8}")
print(f"{'-' * 30}")

total = 0
for name, price, qty in items:
    amount = price * qty
    total += amount
    print(f"{name:<10} {price:>5,}  {qty:>3}개 {amount:>7,}원")

print(f"{'-' * 30}")
print(f"{'합계':>20} {total:>7,}원")
print(f"{'=' * 30}")
```

실행하면 이런 출력이 나와야 합니다.

```
==============================
             영수증             
매장명: 파이썬 편의점
==============================
상품명          단가   수량       금액
------------------------------
삼각김밥      1,200    2개   2,400원
아메리카노    2,500    1개   2,500원
물             800    3개   2,400원
------------------------------
                합계   7,300원
==============================
```

### 실습 2 — 알림 메시지 생성기 만들기

`notifier.py` 파일을 만드세요. 실습 1의 `total` 변수를 활용합니다.

```python
# notifier.py
from string import Template
from datetime import datetime

# 알림 템플릿 (설정 파일에서 읽어왔다고 가정)
sms_template = Template(
    "[$store] 결제 완료!\n금액: $amount원\n일시: $dt"
)

now = datetime.now().strftime("%Y-%m-%d %H:%M")

# 실습 1의 값을 그대로 사용
sms = sms_template.substitute(
    store="파이썬 편의점",
    amount=f"{7300:,}",
    dt=now,
)
print(sms)
```

### 실습 3 — 월별 리포트 함수 만들기

`monthly_report.py` 파일을 만드세요. 실습 1과 2에서 만든 로직을 함수로 묶습니다.

```python
# monthly_report.py
def make_monthly_report(year: int, month: int, sales: list[dict]) -> str:
    total_revenue = sum(s["amount"] for s in sales)
    total_count = sum(s["count"] for s in sales)
    best = max(sales, key=lambda s: s["amount"])

    rows = ""
    for s in sales:
        rows += f"  {s['name']:<10} {s['amount']:>10,}원 ({s['count']:>4}건)\n"

    return f"""
[ {year}년 {month}월 매출 리포트 ]

{rows}
  {'─' * 30}
  총 매출   {total_revenue:>10,}원
  총 건수   {total_count:>10,}건
  최고 상품  {best['name']} ({best['amount']:,}원)
""".strip()


sample_sales = [
    {"name": "아메리카노", "amount": 875000, "count": 350},
    {"name": "카페라떼",   "amount": 720000, "count": 240},
    {"name": "케이크",     "amount": 540000, "count": 108},
]

print(make_monthly_report(2026, 5, sample_sales))
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 원인 | 수정 방법 |
|------|--------------------|------|-----------|
| f-string 안에서 따옴표 충돌 | `SyntaxError: f-string expression part cannot include a backslash` | f-string `{}` 안에서 같은 종류의 따옴표 사용 | 바깥 따옴표와 다른 종류 사용 또는 변수로 분리 |
| `.format()` 인자 개수 불일치 | `IndexError: Replacement index 2 out of range for positional args tuple` | `{}` 개수보다 인자가 적음 | `{}` 개수와 `.format()` 인자 개수를 맞춤 |
| `%` 포맷에서 튜플 누락 | `TypeError: not enough arguments for format string` | 변수가 하나일 때 `% name` 대신 `% (name,)` 로 써야 함 | `% (name,)` 처럼 쉼표를 붙인 튜플로 감쌈 |
| Template에서 변수 누락 | `KeyError: 'menu'` | `substitute()`는 모든 변수가 있어야 함 | `safe_substitute()` 사용 또는 누락 변수 추가 |
| `:,` 를 문자열에 적용 | `ValueError: Cannot specify ',' with 's'` | `,` 포맷은 숫자에만 적용 가능 | 먼저 `int()`/`float()`으로 변환 후 포맷 지정 |
| 정렬 너비가 한글에서 틀어짐 | 출력 정렬이 맞지 않음 | 한글은 글자 너비가 영문의 2배 | `wcwidth` 라이브러리 사용 또는 너비를 조정 |

**자주 나오는 에러 예시:**

```python
# 잘못된 예 — 따옴표 충돌
name = "철수"
# print(f"이름: {"철수"}")  # SyntaxError!

# 올바른 예 1 — 바깥을 작은따옴표로
print(f'이름: {"철수"}')

# 올바른 예 2 — 변수로 분리
label = "철수"
print(f"이름: {label}")
```

---

## 확인 체크리스트

- [ ] `%s`, `%d`, `%f`가 각각 무엇을 의미하는지 설명할 수 있다.
- [ ] `.format()`에서 위치 지정과 이름 지정 방식의 차이를 안다.
- [ ] f-string 앞에 `f`를 붙이는 것을 빠뜨리지 않는다.
- [ ] f-string `{}` 안에 `:`를 써서 포맷을 지정할 수 있다.
- [ ] `:,`로 천 단위 구분자를 넣을 수 있다.
- [ ] `:.2f`로 소수점 자리를 제한할 수 있다.
- [ ] `:<`, `:>`, `:^`로 정렬 방향을 바꿀 수 있다.
- [ ] 여러 줄 f-string을 `"""..."""`으로 작성할 수 있다.
- [ ] `string.Template`이 언제 필요한지 설명할 수 있다.
- [ ] `substitute()`와 `safe_substitute()`의 차이를 안다.
- [ ] 실습 1~3을 오류 없이 실행할 수 있다.

---

## 한 번 더 생각해 보기

1. f-string과 `.format()`은 코드를 실행하는 시점이 다릅니다. 사용자 입력에서 받은 문자열을 f-string으로 직접 사용하면 어떤 위험이 생길 수 있을까요? `string.Template`이 그 대안이 되는 이유를 생각해 보세요.

2. `receipt.py`에서 한글 상품명이 들어갈 때 표 정렬이 맞지 않는 경우가 생깁니다. 한글 한 글자가 영문 두 글자 너비를 차지하기 때문인데, 이 문제를 어떻게 해결하면 좋을지 아이디어를 적어 보세요.

3. 지금까지 배운 세 가지 방식(`%`, `.format()`, f-string) 중 여러분이 앞으로 가장 많이 쓸 것 같은 방식은 무엇인가요? 그 이유는 무엇인가요?

---

## 다음 장

다음 장에서는 파일 읽기와 쓰기를 배웁니다. 이번 장에서 만든 영수증과 리포트를 실제 `.txt` 파일로 저장하는 방법을 익혀 봅니다.