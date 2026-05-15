## 이 장에서 배우는 것

- AI(Claude 등)와 함께 조건문(if/elif/else)을 작성하는 방법
- AI와 함께 반복문(for/while)을 이해하고 디버깅하는 방법
- AI와 함께 함수(def)를 설계하고 개선하는 방법
- AI에게 "제대로 된 질문"을 하는 패턴 익히기
- AI가 틀릴 수 있다는 것을 알고, 직접 실행해서 검증하는 습관 들이기

---

## 먼저 쉬운 설명

코딩을 처음 배울 때 가장 힘든 순간은 **"내가 뭘 모르는지도 모르겠다"**고 느낄 때입니다.

조건문, 반복문, 함수는 파이썬에서 가장 중요한 세 가지 개념입니다. 이 세 가지를 모르면 사실상 프로그램을 만들 수 없어요. 그런데 이 개념들은 처음 보면 머릿속에 잘 그려지지 않습니다.

여기서 AI가 도움이 됩니다. AI는 24시간 내 곁에 있는 친절한 과외 선생님 같아요. "이게 왜 안 되나요?" "더 짧게 쓸 수 있나요?" "이 코드 어떻게 생각해요?" — 이런 질문을 언제든지 할 수 있습니다.

단, **AI가 말한다고 무조건 맞는 게 아닙니다.** AI는 자신 있게 틀린 말을 하기도 합니다. 그래서 AI가 준 코드는 반드시 직접 실행해보는 습관이 중요합니다.

이 장에서는 조건문 → 반복문 → 함수를 배우면서, 각각 AI를 어떻게 활용하면 좋은지 구체적인 대화 예시와 함께 알아봅니다.

---

## 1. 조건문과 AI 함께 쓰기

### 조건문 복습

조건문은 "만약 ~라면 ~하고, 아니라면 ~한다"는 흐름입니다.

```python
# score_check.py
score = int(input("점수를 입력하세요: "))

if score >= 90:
    print("A학점입니다!")
elif score >= 80:
    print("B학점입니다.")
elif score >= 70:
    print("C학점입니다.")
else:
    print("재시험 대상입니다.")
```

### AI에게 조건문을 물어보는 방법

**나쁜 질문 예시:**
> "조건문 알려줘"

→ 너무 막연합니다. AI가 교과서 설명만 돌려줄 가능성이 높아요.

**좋은 질문 예시:**
> "파이썬에서 점수가 90 이상이면 A, 80 이상이면 B, 그 이하면 C를 출력하는 코드를 써줘. 그런데 점수가 0보다 작거나 100보다 크면 '잘못된 점수입니다'라고 출력해줘."

→ 조건을 구체적으로 말하면 AI가 훨씬 정확한 코드를 줍니다.

### AI가 준 코드 직접 검증하기

```python
# ai_score_check.py — AI가 제안한 코드 (검증 전)
def check_grade(score):
    if score < 0 or score > 100:
        print("잘못된 점수입니다.")
    elif score >= 90:
        print("A")
    elif score >= 80:
        print("B")
    else:
        print("C")

check_grade(95)   # A 출력되는지 확인
check_grade(85)   # B 출력되는지 확인
check_grade(-5)   # 잘못된 점수 출력되는지 확인
check_grade(101)  # 잘못된 점수 출력되는지 확인
```

> **AI와 함께 팁:** AI가 코드를 줬을 때, "이 코드에서 빠진 예외 상황이 있을까?"라고 다시 물어보세요. AI 스스로 자신의 코드의 빈틈을 찾아줄 때가 많습니다.

---

## 2. 반복문과 AI 함께 쓰기

### 반복문 복습

반복문은 같은 작업을 여러 번 반복할 때 씁니다.

```python
# multiplication_table.py
number = int(input("몇 단을 출력할까요? "))

for i in range(1, 10):
    print(f"{number} × {i} = {number * i}")
```

### AI에게 반복문 디버깅 요청하기

아래 코드는 오류가 있습니다. 실행해보세요.

```python
# bug_loop.py
fruits = ["사과", "바나나", "오렌지"]

for i in range(4):          # 문제: range(4)는 0,1,2,3 → 인덱스 3은 없음
    print(fruits[i])
```

실행하면 이런 오류가 납니다:
```
IndexError: list index out of range
```

**AI에게 디버깅 요청하는 좋은 방법:**

> "아래 파이썬 코드를 실행했더니 `IndexError: list index out of range` 오류가 났어. 어디가 문제인지, 왜 그 오류가 나는지 설명해줘. 그리고 고친 버전도 알려줘."
> ```python
> fruits = ["사과", "바나나", "오렌지"]
> for i in range(4):
>     print(fruits[i])
> ```

AI는 보통 이렇게 설명해줍니다:
- `fruits` 리스트에 요소가 3개이므로 유효한 인덱스는 `0, 1, 2`입니다.
- `range(4)`는 `0, 1, 2, 3`을 만들어 `i=3`일 때 `fruits[3]`을 찾으려 하는데, 없으므로 오류가 납니다.

**AI가 제안한 수정 코드:**

```python
# fixed_loop.py
fruits = ["사과", "바나나", "오렌지"]

# 방법 1: range를 리스트 길이에 맞추기
for i in range(len(fruits)):
    print(fruits[i])

# 방법 2 (더 파이썬답게): 직접 순회
for fruit in fruits:
    print(fruit)
```

> **AI와 함께 팁:** 오류 메시지 전체를 AI에게 붙여넣으세요. "오류났어요"보다 `IndexError: list index out of range`처럼 실제 오류 메시지를 주면 AI의 답변 정확도가 크게 올라갑니다.

### while 반복문도 확인하기

```python
# countdown.py
count = 5

while count > 0:
    print(f"{count}...")
    count -= 1   # 이 줄이 없으면 무한 루프!

print("발사!")
```

**AI에게 무한 루프 관련 질문 예시:**
> "while 반복문에서 무한 루프가 생기는 패턴을 3가지 알려줘. 각각 예시 코드와 함께."

---

## 3. 함수와 AI 함께 쓰기

### 함수 복습

함수는 "이름을 붙인 코드 묶음"입니다. 한번 만들어두면 여러 번 재사용할 수 있어요.

```python
# greeting.py
def greet(name, time_of_day):
    """이름과 시간대를 받아 인사말을 출력한다."""
    print(f"좋은 {time_of_day}, {name}님!")

greet("지수", "아침")
greet("민준", "저녁")
```

### AI와 함께 함수 설계하기

큰 코드를 짜기 전에 AI에게 설계를 물어볼 수 있습니다.

**AI에게 함수 설계 요청 예시:**
> "간단한 계산기 프로그램을 만들려고 해. 더하기, 빼기, 곱하기, 나누기 기능이 필요해. 이걸 함수로 어떻게 나누면 좋을지 설계를 알려줘."

AI가 제안할 수 있는 구조:

```python
# calculator.py

def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        return "0으로 나눌 수 없습니다."
    return a / b

def calculate(a, operator, b):
    if operator == "+":
        return add(a, b)
    elif operator == "-":
        return subtract(a, b)
    elif operator == "*":
        return multiply(a, b)
    elif operator == "/":
        return divide(a, b)
    else:
        return "알 수 없는 연산자입니다."

# 테스트
print(calculate(10, "+", 5))   # 15
print(calculate(10, "/", 0))   # 0으로 나눌 수 없습니다.
```

### AI에게 코드 리뷰 요청하기

코드를 직접 짠 다음, AI에게 "이 코드 어때요?"라고 물어보는 습관을 들이세요.

**AI에게 코드 리뷰 요청 예시:**
> "아래 코드를 짰는데 개선할 점이 있으면 알려줘. 특히 초보자가 자주 하는 실수가 있는지 봐줘."

```python
# my_calculator.py — 내가 직접 짠 코드
def calc(a, op, b):
    if op == "+": print(a+b)
    if op == "-": print(a-b)
    if op == "*": print(a*b)
    if op == "/": print(a/b)
```

AI가 지적할 수 있는 것들:
- `if`를 연속으로 쓰면 모든 조건을 다 검사합니다. `elif`를 쓰는 게 더 효율적입니다.
- `b == 0`일 때 나누기 처리가 없어서 `ZeroDivisionError`가 납니다.
- 함수가 `print`로 출력만 하고 값을 `return`하지 않아서 결과를 재사용할 수 없습니다.

> **AI와 함께 팁:** "내 코드에서 오류가 나는 경우를 억지로 만들어줘"라고 AI에게 부탁해보세요. 어떤 입력을 주면 코드가 망가지는지 AI가 테스트 케이스를 만들어줍니다.

---

## 따라 하기 실습

### 실습 1: AI에게 조건문 개선 요청하기

1. 아래 파일을 `grade_checker.py`라는 이름으로 저장하세요.

```python
# grade_checker.py
score = int(input("점수 입력: "))
if score >= 90:
    print("A")
if score >= 80:
    print("B")
if score >= 70:
    print("C")
if score < 70:
    print("F")
```

2. 직접 실행해서 점수 `85`를 입력해보세요. 예상과 다른 결과가 나올 겁니다.

3. AI에게 다음과 같이 질문하세요:
   > "아래 코드를 실행하면 점수가 85일 때 B와 C가 둘 다 출력됩니다. 왜 그런지 설명해주고, if를 elif로 고쳐줘."

4. AI가 준 수정 코드를 `grade_checker_v2.py`로 저장한 후 직접 실행해서 `85`, `92`, `75`, `60`을 각각 입력해 확인하세요.

---

### 실습 2: AI와 함께 반복문 짜기

1. 아래 요구사항을 그대로 복사해서 AI에게 보내세요.
   > "파이썬으로 `number_game.py` 파일을 만들려고 해. 프로그램이 1~10 사이 숫자를 하나 정하고, 사용자가 맞힐 때까지 계속 입력받는 거야. 너무 크다/너무 작다 힌트를 줘야 해. while 반복문을 써줘."

2. AI가 준 코드를 `number_game.py`로 저장하고 직접 실행하세요.

3. 잘 동작하면 AI에게 이어서 질문하세요:
   > "방금 만든 number_game.py에 '몇 번 만에 맞혔는지' 출력하는 기능을 추가해줘."

---

### 실습 3: 함수를 쪼개서 계산기 완성하기

1. 실습 1, 2에서 배운 내용을 활용해서 `mini_calculator.py`를 직접 작성해보세요. 규칙:
   - `add`, `subtract`, `multiply`, `divide` 함수를 각각 만들 것
   - 사용자에게 숫자 두 개와 연산자를 입력받을 것
   - 잘못된 연산자를 입력하면 안내 메시지를 출력할 것

2. 다 짠 후에 AI에게 붙여넣고 이렇게 질문하세요:
   > "이 코드에 버그가 생기는 입력 케이스를 3개 만들어줘. 각각 어떤 오류가 나는지도 알려줘."

3. AI가 알려준 버그를 직접 수정하고 `mini_calculator_final.py`로 저장하세요.

---

## 자주 하는 실수

| 상황 | 오류 메시지 또는 증상 | 원인 | 해결 방법 |
|---|---|---|---|
| `if` 연속 사용 | 조건이 여러 개 동시에 출력됨 | `if`는 모든 조건을 독립 검사 | `elif`로 바꾸기 |
| 리스트 인덱스 초과 | `IndexError: list index out of range` | `range()` 숫자가 리스트 길이보다 큼 | `range(len(리스트))` 또는 `for item in 리스트` 사용 |
| 무한 루프 | 프로그램이 멈추지 않음 | `while` 안에서 조건 변수를 바꾸지 않음 | 루프 안에서 카운터 증가/감소 확인 |
| 0으로 나누기 | `ZeroDivisionError: division by zero` | 나누는 값이 0인지 검사 안 함 | `if b == 0:` 분기 추가 |
| 함수 반환값 없음 | `None`이 출력됨 | `print`는 했지만 `return`을 안 함 | `return` 값 추가 또는 의도 확인 |
| 들여쓰기 오류 | `IndentationError: expected an indented block` | 함수/조건문 아래 코드 들여쓰기 안 함 | 스페이스 4칸 또는 탭으로 들여쓰기 |
| AI 코드 그대로 붙여넣기 | 실행은 되지만 예상과 다른 동작 | AI가 요구사항을 잘못 이해했거나 엣지 케이스 누락 | 직접 여러 입력값으로 테스트 |

---

## 확인 체크리스트

- [ ] `if`, `elif`, `else`의 차이를 말로 설명할 수 있다.
- [ ] `for`와 `while`을 언제 쓰는지 구분할 수 있다.
- [ ] 함수를 `def`로 직접 정의하고 호출해봤다.
- [ ] `return`과 `print`의 차이를 안다.
- [ ] AI에게 오류 메시지 전체를 붙여넣어 디버깅을 요청해봤다.
- [ ] AI가 준 코드를 그냥 쓰지 않고 직접 실행해서 검증했다.
- [ ] AI에게 코드 리뷰를 요청해봤다.
- [ ] `IndexError`, `ZeroDivisionError`, `IndentationError` 오류를 실제로 만나고 고쳐봤다.

---

## 한 번 더 생각해 보기

1. AI에게 코드를 부탁할 때 "조건문 알려줘"와 "점수가 90 이상이면 A, 80 이상이면 B를 출력하는 코드를 써줘"는 결과가 왜 다를까요? 구체적인 질문이 왜 더 좋은지 자신의 말로 설명해보세요.

2. AI가 짜준 코드가 실행은 됐지만 결과가 이상했던 경험이 있나요? (혹은 상상해보세요.) AI를 믿되 검증해야 하는 이유를 한 문장으로 말해보세요.

3. 조건문, 반복문, 함수 — 이 세 가지를 모두 써서 만들 수 있는 간단한 프로그램을 하나 떠올려보세요. AI에게 설계를 물어보기 전에 먼저 종이에 흐름을 그려보면 어떨까요?

---

## 다음 장

다음 장에서는 클래스와 객체(class & object)를 배우고, AI와 함께 현실적인 미니 프로젝트를 설계하는 방법을 익힙니다.