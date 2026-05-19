## 이 장에서 배우는 것

- AI가 생성한 코드에서 버그가 왜 생기는지 이해한다
- 오류 메시지를 읽고 무슨 뜻인지 파악하는 방법을 익힌다
- 디버깅의 기본 흐름(읽기 → 찾기 → 고치기 → 확인하기)을 직접 실습한다
- AI에게 버그를 설명하고 수정을 요청하는 올바른 방법을 배운다
- 흔히 반복되는 AI 버그 패턴 5가지를 인식하고 스스로 고칠 수 있다

---

## 먼저 쉬운 설명

AI는 신기할 만큼 빠르게 코드를 써줍니다. 하지만 AI가 쓴 코드가 항상 바로 동작하지는 않아요. 마치 새로 입사한 똑똑한 동료가 우리 팀의 규칙을 아직 모르는 것처럼, AI도 **여러분의 프로젝트 맥락**을 완벽하게 이해하지 못합니다.

그래서 AI 코드를 그냥 붙여넣기만 하면 위험합니다. 오류가 나도 왜 나는지 모르면, 다시 AI에게 물어봐도 같은 실수가 반복됩니다.

이 장을 마치면 **오류 메시지를 무서워하지 않게** 됩니다. 오류 메시지는 "코드가 망했어요"가 아니라 "여기 이 부분이 문제예요, 확인해봐요"라고 알려주는 친절한 안내판이거든요.

---

## 1. 오류 메시지 읽는 법

오류 메시지는 아래 세 가지 정보를 항상 담고 있습니다.

| 위치 | 줄 번호 (line number) |
| 종류 | 어떤 종류의 오류인가 (TypeError, NameError 등) |
| 이유 | 왜 오류가 났는가 (한 줄 설명) |

**예시: AI가 만들어준 회원 나이 계산 코드**

```python
# user_age.py

def calculate_age(birth_year):
    current_year = 2026
    age = current_year - birth_year
    return age

user_input = input("태어난 연도를 입력하세요: ")
print("당신의 나이는", calculate_age(user_input), "세입니다.")
```

실행하면 이런 오류가 납니다:

```
Traceback (most recent call last):
  File "user_age.py", line 9, in <module>
    print("당신의 나이는", calculate_age(user_input), "세입니다.")
  File "user_age.py", line 4, in calculate_age
    age = current_year - birth_year
TypeError: unsupported operand type(s) for -: 'int' and 'str'
```

읽는 순서:
1. **마지막 줄**부터 읽는다 → `TypeError: unsupported operand type(s) for -: 'int' and 'str'`
2. **File 줄**을 본다 → `user_age.py` 4번째 줄
3. **문제 코드**를 확인한다 → `age = current_year - birth_year`

해석: `current_year`는 숫자(int)인데 `birth_year`는 문자열(str)이라서 빼기 연산이 안 된다는 뜻입니다.

**수정:**

```python
# user_age.py (수정 버전)

def calculate_age(birth_year):
    current_year = 2026
    age = current_year - birth_year
    return age

user_input = input("태어난 연도를 입력하세요: ")
print("당신의 나이는", calculate_age(int(user_input)), "세입니다.")  # int() 추가
```

---

## 2. AI 버그의 5가지 대표 패턴

AI가 자주 저지르는 실수에는 패턴이 있습니다. 이 패턴을 알면 코드를 보자마자 의심해야 할 부분을 빠르게 찾을 수 있습니다.

### 패턴 1: 타입 불일치 (Type Mismatch)

위에서 본 것처럼 숫자와 문자열을 섞어 쓰는 실수입니다.

```python
# order_summary.py

def print_order(item_name, quantity, price):
    total = quantity * price
    print(item_name + "을(를) " + quantity + "개 주문했습니다. 총액: " + str(total) + "원")

print_order("노트북", 2, 1500000)
```

오류:
```
TypeError: can only concatenate str (not "int") to str
```

수정:
```python
def print_order(item_name, quantity, price):
    total = quantity * price
    print(item_name + "을(를) " + str(quantity) + "개 주문했습니다. 총액: " + str(total) + "원")
```

---

### 패턴 2: 존재하지 않는 변수 (NameError)

AI가 변수 이름을 잘못 짓거나, 아직 정의되지 않은 변수를 사용하는 경우입니다.

```python
# score_checker.py

def check_pass(score):
    if score >= 60:
        result = "합격"
    print("결과:", ressult)  # 오타: ressult

check_pass(75)
```

오류:
```
NameError: name 'ressult' is not defined
```

수정:
```python
def check_pass(score):
    if score >= 60:
        result = "합격"
    else:
        result = "불합격"
    print("결과:", result)
```

---

### 패턴 3: 들여쓰기 오류 (IndentationError)

AI가 코드를 복사하거나 조합할 때 들여쓰기가 어긋나는 경우입니다.

```python
# greet_user.py

def greet(name):
    message = "안녕하세요, " + name + "님!"
        print(message)  # 들여쓰기가 한 칸 더 들어가 있음

greet("지수")
```

오류:
```
IndentationError: unexpected indent
```

수정:
```python
def greet(name):
    message = "안녕하세요, " + name + "님!"
    print(message)  # 들여쓰기를 함수 본문과 맞춤
```

---

### 패턴 4: 없는 키 접근 (KeyError)

딕셔너리에서 없는 키를 꺼내려 할 때 납니다. AI가 예시 데이터와 다른 키 이름을 쓸 때 자주 발생합니다.

```python
# member_info.py

member = {
    "name": "김철수",
    "age": 28,
    "email": "chulsoo@example.com"
}

print("전화번호:", member["phone"])  # phone 키가 없음
```

오류:
```
KeyError: 'phone'
```

수정:
```python
print("전화번호:", member.get("phone", "등록되지 않음"))  # .get() 사용
```

---

### 패턴 5: 임포트 누락 (ModuleNotFoundError / ImportError)

AI가 표준 라이브러리나 외부 패키지를 사용하면서 `import` 문을 빠뜨리는 경우입니다.

```python
# date_formatter.py

def today_string():
    today = datetime.date.today()  # datetime을 import하지 않음
    return today.strftime("%Y년 %m월 %d일")

print(today_string())
```

오류:
```
NameError: name 'datetime' is not defined
```

수정:
```python
import datetime  # 파일 맨 위에 추가

def today_string():
    today = datetime.date.today()
    return today.strftime("%Y년 %m월 %d일")
```

---

## 3. AI에게 버그 수정 요청하는 올바른 방법

AI에게 막연하게 "왜 안 돼요?"라고 물으면 엉뚱한 답이 나올 수 있습니다. 아래 템플릿을 사용하면 훨씬 정확한 답을 받을 수 있습니다.

**나쁜 요청 예시:**
```
이 코드 왜 오류 나요?
```

**좋은 요청 템플릿:**
```
아래 코드를 실행하면 오류가 납니다.

[코드 전체 붙여넣기]

오류 메시지:
[오류 메시지 전체 붙여넣기]

제가 하려는 것:
[목표를 한 문장으로 설명]

어디가 문제인지, 어떻게 고치면 되는지 알려주세요.
```

**실제 예시:**

```
아래 코드를 실행하면 오류가 납니다.

# score_checker.py
def check_pass(score):
    if score >= 60:
        result = "합격"
    print("결과:", ressult)

check_pass(75)

오류 메시지:
NameError: name 'ressult' is not defined

제가 하려는 것:
점수를 입력받아 합격/불합격을 출력하고 싶습니다.

어디가 문제인지, 어떻게 고치면 되는지 알려주세요.
```

---

## 따라 하기 실습

### 실습 1: 오류 메시지 직접 분석하기

아래 파일을 `product_discount.py`로 저장하고 실행해 보세요.

```python
# product_discount.py

def apply_discount(price, discount_rate):
    discounted = price * (1 - discount_rate)
    return discounted

products = [
    {"name": "티셔츠", "price": "29000"},
    {"name": "청바지", "price": 59000},
]

for product in products:
    final_price = apply_discount(product["price"], 0.1)
    print(product["name"], "할인가:", final_price, "원")
```

1. 코드를 실행합니다.
2. 오류 메시지 전체를 노트에 적거나 복사합니다.
3. 오류 메시지에서 **줄 번호**, **오류 종류**, **이유** 세 가지를 찾아서 직접 써봅니다.
4. 이 장에서 배운 패턴 중 어떤 패턴인지 이름을 맞춰봅니다.

---

### 실습 2: 버그 고치기

실습 1에서 찾은 버그를 직접 수정해 보세요.

- `"29000"` 이 문자열인지 숫자인지 확인합니다.
- `int()` 또는 데이터 자체를 수정하여 오류 없이 실행되게 합니다.
- 두 제품 모두 할인가가 올바르게 출력되면 성공입니다.

기대 출력:
```
티셔츠 할인가: 26100.0 원
청바지 할인가: 53100.0 원
```

---

### 실습 3: AI에게 수정 요청하고 검증하기

아래 코드를 `member_register.py`로 저장하고 실습합니다.

```python
# member_register.py

def register_member(name, age, email):
    member = {
        "이름": name,
        "나이": age,
        "이메일": email,
        "가입일": today_str()  # 이 함수가 없음
    }
    return member

new_member = register_member("박민준", 25, "minjun@example.com")
print(new_member)
```

1. 코드를 실행하여 오류를 확인합니다.
2. 위에서 배운 **좋은 요청 템플릿**을 사용해 AI에게 수정을 요청합니다.
3. AI가 준 수정 코드를 그냥 붙여넣지 말고, 고친 부분이 무엇인지 직접 확인합니다.
4. 수정된 코드를 실행하여 가입일이 포함된 딕셔너리가 출력되는지 확인합니다.

---

## 자주 하는 실수

| 실수 | 오류 메시지 | 원인 | 수정 방법 |
|------|-------------|------|-----------|
| 문자열과 숫자를 더함 | `TypeError: can only concatenate str (not "int") to str` | `input()`은 항상 문자열 반환 | `int()` 또는 `str()`로 타입 변환 |
| 변수 이름 오타 | `NameError: name 'xxx' is not defined` | AI가 다른 이름 사용 또는 오타 | 오류 줄 번호의 변수명 확인 후 통일 |
| 들여쓰기 불일치 | `IndentationError: unexpected indent` | AI 코드 조합 시 들여쓰기 어긋남 | 탭/스페이스 혼용 여부 확인, 일관되게 수정 |
| 딕셔너리 키 없음 | `KeyError: 'xxx'` | AI가 다른 키 이름을 가정 | `.get(키, 기본값)` 사용 또는 키 이름 확인 |
| 라이브러리 미임포트 | `NameError` 또는 `ModuleNotFoundError` | AI가 import 문 누락 | 파일 맨 위에 `import 모듈명` 추가 |
| 리스트 범위 초과 | `IndexError: list index out of range` | AI가 리스트 크기를 잘못 가정 | 인덱스 값과 리스트 길이(`len()`) 비교 확인 |

---

## 확인 체크리스트

- [ ] 오류 메시지에서 줄 번호를 찾을 수 있다
- [ ] `TypeError`, `NameError`, `IndentationError`, `KeyError`, `ImportError` 가 각각 무슨 뜻인지 설명할 수 있다
- [ ] AI 코드를 받았을 때 바로 붙여넣지 않고 타입, 변수명, import 세 가지를 먼저 확인한다
- [ ] AI에게 버그 수정을 요청할 때 오류 메시지 전체를 함께 제공한다
- [ ] `.get()` 을 사용해 KeyError 없이 딕셔너리에서 값을 꺼낼 수 있다
- [ ] 실습 3에서 AI가 고친 코드의 **변경 내용을 직접 확인**하고 실행했다

---

## 한 번 더 생각해 보기

1. AI가 생성한 코드를 그냥 실행했는데 오류 없이 잘 돌아갔습니다. 이 코드를 믿어도 될까요? 오류가 없다는 것이 코드가 올바르다는 것과 같은 의미일까요?

2. 같은 오류가 며칠 동안 계속 반복해서 생긴다면, 매번 AI에게 물어보는 것이 최선일까요? 더 좋은 방법이 있다면 무엇일지 생각해 보세요.

3. AI에게 "이 코드 틀렸어, 다시 써줘"라고 요청하는 것과 "3번째 줄에서 TypeError가 납니다. 오류 메시지는 이것이고, 제 목표는 이것입니다"라고 요청하는 것 중 왜 후자가 더 좋은 결과를 만들어낼까요?

---

## 다음 장

다음 장에서는 버그를 고치는 것을 넘어, AI와 함께 처음부터 버그가 적은 코드를 작성하는 방법인 **테스트 코드 작성과 pytest 기초**를 배웁니다.