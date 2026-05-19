## 이 장에서 배우는 것

- AI가 생성한 Python 코드 초안에서 오류를 찾는 방법
- 오류를 수정하고 그 이유를 설명하는 능력
- 흔한 Python 문법 오류와 논리 오류의 차이
- 수정한 코드를 검증하는 체계적인 방법
- AI 도구를 보조 수단으로 활용하되, 스스로 판단하는 습관

---

## 먼저 쉬운 설명

AI는 코드를 빠르게 만들어 주지만, **항상 완벽하지는 않습니다.**

마치 선배가 써 준 메모처럼, 방향은 맞을 수 있지만 세부적인 오탈자나 논리 실수가 있을 수 있습니다. 이 장에서는 AI가 만든 코드 초안을 받아서, 오류를 직접 찾고 고치는 연습을 합니다.

이 능력은 실제 개발 현장에서도 매우 중요합니다. 코드를 처음부터 짜는 것뿐 아니라, **다른 사람(또는 AI)이 쓴 코드를 읽고 판단하는 것**도 개발자의 핵심 역량입니다.

> 이 수업은 약 1시간 기준으로 설계되었습니다.
> - 개념 학습: 15분
> - 실습 1–3: 30분
> - 체크리스트 & 복습: 15분

---

## 1. 문법 오류(Syntax Error)란?

Python은 규칙을 엄격하게 지킵니다. 철자 하나만 틀려도 실행 자체가 되지 않습니다.

**AI 초안 (broken draft):**

```python
# 파일명: greeting.py
# AI가 생성한 초안 — 오류 있음

def greet(name)
    print("안녕하세요, " + name + "!")

greet("지수")
```

**오류 메시지:**

```
  File "greeting.py", line 2
    def greet(name)
                   ^
SyntaxError: expected ':'
```

**수정 포인트:**

`def` 함수 정의 줄 끝에는 반드시 콜론(`:`)이 있어야 합니다.

**수정 후:**

```python
# 파일명: greeting.py

def greet(name):
    print("안녕하세요, " + name + "!")

greet("지수")
```

---

## 2. 들여쓰기 오류(Indentation Error)란?

Python은 들여쓰기로 코드 블록을 구분합니다. 스페이스가 하나만 달라도 오류가 납니다.

**AI 초안 (broken draft):**

```python
# 파일명: score_check.py

def check_pass(score):
    if score >= 60:
        print("합격입니다!")
      print("수고하셨습니다.")  # 들여쓰기가 맞지 않음

check_pass(75)
```

**오류 메시지:**

```
  File "score_check.py", line 6
    print("수고하셨습니다.")
                         ^
IndentationError: unindent does not match any outer indentation level
```

**수정 포인트:**

`if` 블록 안의 코드는 모두 같은 깊이로 들여써야 합니다. `print("수고하셨습니다.")` 는 `if` 블록 밖으로 내보내거나, 안으로 맞춰야 합니다.

**수정 후 (if 블록 밖으로 이동):**

```python
# 파일명: score_check.py

def check_pass(score):
    if score >= 60:
        print("합격입니다!")
    print("수고하셨습니다.")  # if와 동일한 들여쓰기 수준

check_pass(75)
```

---

## 3. 논리 오류(Logic Error)란?

문법은 맞지만 프로그램이 **의도한 대로 동작하지 않는** 경우입니다. 가장 찾기 어려운 오류입니다.

**AI 초안 (broken draft):**

```python
# 파일명: discount.py
# 1만 원 이상 구매 시 10% 할인

def apply_discount(price):
    if price > 10000:        # 오류: 10000원 정확히는 할인 안 됨
        discounted = price * 0.10  # 오류: 할인 금액을 계산하는 것인지, 최종 가격인지 불명확
        return discounted

result = apply_discount(10000)
print("최종 가격:", result)
```

**실행 결과 (오류 메시지 없음, 하지만 결과가 이상함):**

```
최종 가격: None
```

**수정 포인트:**

1. `price > 10000` 조건은 정확히 10,000원일 때 할인을 적용하지 않습니다. `>=` 로 바꿔야 합니다.
2. `price * 0.10` 은 할인 금액(1,000원)이지, 최종 가격(9,000원)이 아닙니다.
3. `price < 10000` 인 경우 `return` 이 없으므로 `None` 이 반환됩니다.

**수정 후:**

```python
# 파일명: discount.py

def apply_discount(price):
    if price >= 10000:
        discounted = price * 0.90  # 90%가 최종 가격
        return discounted
    return price  # 할인 없을 때는 원래 가격 반환

result = apply_discount(10000)
print("최종 가격:", result)
```

**실행 결과:**

```
최종 가격: 9000.0
```

---

## 4. 타입 오류(Type Error)란?

Python에서 숫자와 문자열을 직접 합치려 하면 오류가 납니다.

**AI 초안 (broken draft):**

```python
# 파일명: bill.py

item = "커피"
price = 4500
count = 2

print("주문: " + item + " × " + count + "잔 = " + price * count + "원")
```

**오류 메시지:**

```
TypeError: can only concatenate str (not "int") to str
```

**수정 포인트:**

숫자를 문자열과 `+` 로 이을 때는 `str()` 로 변환하거나, `f-string` 을 사용합니다.

**수정 후 (f-string 사용):**

```python
# 파일명: bill.py

item = "커피"
price = 4500
count = 2

print(f"주문: {item} × {count}잔 = {price * count}원")
```

**실행 결과:**

```
주문: 커피 × 2잔 = 9000원
```

---

## 따라 하기 실습

### 실습 1 — 문법·들여쓰기 오류 수정하기

아래 파일을 `temperature.py` 로 저장하고, 오류를 찾아 모두 수정하세요.

```python
# temperature.py (broken draft)

def celsius_to_fahrenheit(c)
    result = c * 9/5 + 32
  return result

temp = celsius_to_fahrenheit(100)
print("화씨:", temp)
```

**힌트:** 오류가 두 군데 있습니다. 하나는 문법 오류, 하나는 들여쓰기 오류입니다.

**예상 실행 결과:**

```
화씨: 212.0
```

---

### 실습 2 — 논리 오류 수정하기

`실습 1` 을 완료한 후, 아래 파일을 `grade.py` 로 저장하세요. 실행은 되지만 결과가 틀립니다. 무엇이 잘못됐는지 설명하고 수정하세요.

```python
# grade.py (broken draft)

def get_grade(score):
    if score >= 90:
        return "A"
    if score >= 80:
        return "B"
    if score >= 70:
        return "C"
    if score >= 60:
        return "D"
    if score >= 0:
        return "F"  # 오류: 0점 이상이면 모두 F?

print(get_grade(55))
print(get_grade(72))
```

**수정 포인트를 한 문장으로 적어 보세요.**

**수정 후 예상 결과:**

```
F
C
```

---

### 실습 3 — 타입 오류 수정 + 기능 확장하기

`실습 2` 를 완료한 후, 아래 파일을 `report_card.py` 로 저장하세요. 오류를 수정하고, 학생 이름도 함께 출력되도록 코드를 추가하세요.

```python
# report_card.py (broken draft)

def print_result(name, score):
    grade = "A" if score >= 90 else "B" if score >= 80 else "C"
    print("학생: " + name + " | 점수: " + score + " | 등급: " + grade)

print_result("민준", 85)
```

**수정 후 예상 결과:**

```
학생: 민준 | 점수: 85 | 등급: B
```

---

## 자주 하는 실수

| 실수 유형 | 잘못된 코드 예시 | 오류 메시지 | 수정 방법 |
|---|---|---|---|
| 콜론 빠뜨림 | `def greet(name)` | `SyntaxError: expected ':'` | 함수·조건문 끝에 `:` 추가 |
| 들여쓰기 불일치 | 스페이스 2개 + 스페이스 4개 혼용 | `IndentationError` | 스페이스 4개로 통일 (또는 탭 통일) |
| 숫자+문자열 연결 | `"점수: " + 85` | `TypeError: can only concatenate str` | `str(85)` 또는 f-string 사용 |
| 조건 범위 오류 | `>` 와 `>=` 혼동 | 오류 없음, 결과만 틀림 | 경계값을 직접 테스트해서 확인 |
| `return` 누락 | 함수 안에 `return` 없음 | 오류 없음, `None` 반환 | 모든 경우에 `return` 있는지 확인 |
| `=` 와 `==` 혼동 | `if score = 100:` | `SyntaxError: invalid syntax` | 비교는 `==`, 대입은 `=` |

---

## 확인 체크리스트

수정한 코드를 제출하기 전에 아래 항목을 스스로 점검하세요.

- [ ] 모든 함수 정의(`def`) 줄 끝에 콜론(`:`)이 있다
- [ ] 들여쓰기가 스페이스 4칸으로 일관되게 맞춰져 있다
- [ ] 숫자를 문자열과 연결할 때 `str()` 또는 f-string을 사용했다
- [ ] 조건문의 `>` / `>=` 경계값을 실제로 입력해서 테스트했다
- [ ] 함수가 모든 경우에 값을 `return` 하는지 확인했다
- [ ] 코드를 실행했을 때 예상 결과와 실제 결과가 일치한다
- [ ] 수정한 이유를 한 문장으로 설명할 수 있다
- [ ] AI 초안과 수정본의 차이를 동료에게 말로 설명할 수 있다

---

## 한 번 더 생각해 보기

1. **AI가 만든 코드를 그냥 복사해서 쓰면 어떤 문제가 생길 수 있을까요?** 이번 실습에서 발견한 오류들을 바탕으로, 검토 없이 사용했을 때 어떤 결과가 나왔을지 생각해 보세요.

2. **논리 오류는 왜 문법 오류보다 찾기 어려울까요?** Python이 오류 메시지를 보여 주지 않는데 어떻게 논리 오류를 발견할 수 있을지, 자신만의 방법을 떠올려 보세요.

3. **코드를 수정한 후 "올바르게 고쳤다"는 것을 어떻게 확인할 수 있을까요?** 하나의 테스트 값만 확인하는 것으로 충분할까요? 어떤 값들을 추가로 테스트해야 할지 생각해 보세요.

---

## 다음 장

다음 장에서는 Python 함수를 더 체계적으로 설계하고, 여러 오류 상황을 `try / except` 로 안전하게 처리하는 방법을 배웁니다.