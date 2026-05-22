# Chapter 02: 변수 — 값에 이름을 붙이는 방법

## 이 장에서 배우는 것

- 변수가 왜 필요한지 (같은 값을 여러 번 쓸 때, 나중에 바꿔야 할 때)
- 변수를 만들고 값을 저장하는 방법
- 변수 이름을 짓는 규칙 (snake_case, 예약어 금지)
- 변수 값을 바꾸는 방법, 여러 변수를 한 번에 할당하는 방법
- 자주 쓰는 패턴 (카운터, 누산기, 임시 저장)
- 변수 타입 확인 (`type()`)
- 자주 겪는 실수 (NameError, 대소문자 실수)
- 실습 3개: 학생 정보 저장, 계산기 만들기, 변수 값 교환

---

## 왜 변수가 필요한가?

처음 Python을 배울 때 변수가 왜 필요한지 잘 와닿지 않는 경우가 많다. 그냥 `print("Mina")` 하면 되는 거 아닌가? 싶기도 하다.

두 가지 상황을 생각해보자.

### 상황 1: 같은 값을 여러 번 써야 할 때

학생의 이름을 10군데 출력해야 하는 프로그램이 있다고 하자.

**변수 없이:**
```python
print("이름: Mina")
print("반장: Mina")
print("수상자: Mina")
print("출석 확인: Mina")
print("성적표 이름: Mina")
```

나중에 이름이 바뀌면? `"Mina"`가 들어간 모든 줄을 찾아서 하나하나 고쳐야 한다. 10개면 그나마 낫지, 100개라면?

**변수 사용:**
```python
name = "Mina"

print("이름:", name)
print("반장:", name)
print("수상자:", name)
print("출석 확인:", name)
print("성적표 이름:", name)
```

이름을 바꾸려면 첫 줄 `name = "Mina"` 한 줄만 수정하면 된다. 나머지는 자동으로 바뀐다.

### 상황 2: 계산 결과를 나중에 써야 할 때

세금 계산을 한 번만 하고, 그 결과를 여러 곳에서 쓰고 싶을 때:

```python
# 변수 없이: 같은 계산을 3번 반복
print("할인 전 세금:", 50000 * 0.1)
print("할인 후 세금:", 50000 * 0.1 * 0.9)
print("최종 금액:", 50000 + 50000 * 0.1)
```

이 방식은 50000이 바뀌면 3군데를 다 고쳐야 한다. 그리고 한 군데라도 빠뜨리면 계산이 틀린다.

```python
# 변수 사용: 한 번 계산하고 이름으로 가져다 씀
price = 50000
tax_rate = 0.1
tax = price * tax_rate

print("할인 전 세금:", tax)
print("할인 후 세금:", tax * 0.9)
print("최종 금액:", price + tax)
```

`price`가 바뀌면 첫 줄만 수정하면 된다. 나머지는 자동으로 반영된다.

**변수는 값을 이름 붙여 저장해두는 상자다.** 상자에 이름을 붙여두면 나중에 그 이름으로 꺼내 쓸 수 있다.

---

## 1. 변수 만들기

변수는 `이름 = 값` 형태로 만든다.

```python
name = "Mina"
age = 14
height = 162.5
is_student = True
```

- `name`은 변수 이름이다
- `=`는 오른쪽 값을 왼쪽 이름에 **저장하라**는 의미다
- `"Mina"`는 저장하는 값이다

수학의 `=` (같다)와 다르다. Python의 `=`는 "저장하라"는 뜻이다. 그래서 아래 코드가 성립한다.

```python
x = 5
x = x + 1   # 수학에서는 말이 안 되지만, Python에서는 "x에 1을 더한 값을 다시 x에 저장"
print(x)    # 6
```

---

## 2. 변수에 저장된 값 출력하기

`print()` 안에 변수 이름을 쓰면 저장된 값이 출력된다.

```python
name = "Mina"
print(name)
```

출력:
```
Mina
```

따옴표 없이 `name`을 쓴다는 점이 중요하다. 따옴표로 감싸면 `name`이라는 글자 자체가 출력된다.

```python
print(name)    # Mina 출력 (변수 값)
print("name")  # name 이라는 글자 출력 (문자열)
```

### 변수와 글자를 함께 출력하기

**방법 1: 콤마로 구분**
```python
name = "Mina"
age = 14
print("이름:", name)
print("나이:", age)
```

출력:
```
이름: Mina
나이: 14
```

콤마로 연결하면 자동으로 공백 하나가 들어간다.

**방법 2: f-string (더 깔끔하다)**
```python
name = "Mina"
age = 14
print(f"이름: {name}, 나이: {age}")
```

출력:
```
이름: Mina, 나이: 14
```

f-string은 `f"..."` 형태로 쓰고, 중괄호 `{}` 안에 변수 이름을 넣으면 값이 자동으로 들어간다. 자주 쓰이는 방법이니 익숙해지면 좋다.

---

## 3. 변수 이름 짓는 규칙

변수 이름은 아무렇게나 짓지 않는다. 규칙이 있다.

### 반드시 지켜야 하는 규칙 (어기면 오류)

- 영문자, 숫자, 밑줄(`_`)만 사용한다
- 숫자로 시작하면 안 된다
- 공백을 쓸 수 없다 (공백 대신 `_` 사용)
- 대소문자를 구분한다 (`name`과 `Name`은 다른 변수)

```python
# 오류가 나는 이름 예시
1score = 90         # SyntaxError: 숫자로 시작
student name = "Kim"  # SyntaxError: 공백 포함
my-score = 80       # SyntaxError: 하이픈 사용 불가
```

### Python 예약어 (keyword) 사용 금지

Python이 이미 특별한 용도로 사용하는 단어들이다. 변수 이름으로 쓸 수 없다.

```python
# 사용하면 안 되는 예약어들
if, else, for, while, def, class, return, import, True, False, None, and, or, not, in, is
```

```python
# 잘못된 예시
if = 10       # SyntaxError
return = "결과"  # SyntaxError
```

예약어를 실수로 쓰면 `SyntaxError`가 난다. 이럴 때는 이름에 설명을 더 붙이면 된다.

```python
# 올바른 대안
if_value = 10
return_value = "결과"
```

### 권장 스타일: snake_case

단어가 여러 개일 때 밑줄로 연결하는 방식을 `snake_case`라고 한다. Python에서 변수 이름에 권장하는 스타일이다.

```python
# 좋은 이름 예시 (snake_case)
student_name = "Jisoo"
total_score = 100
item_count = 5
is_logged_in = True

# 나쁜 이름 예시
s = "Kim"           # 너무 짧아서 의미를 알 수 없음
studentName = "Kim" # camelCase (Python에서는 잘 안 씀)
STUDENTNAME = "Kim" # 대문자는 상수에 쓰는 관례
```

이름은 짧아도 안 되고 너무 길어도 불편하다. **무엇을 담는 변수인지 바로 알 수 있는 이름**이 가장 좋다.

---

## 4. 변수 값 바꾸기

변수는 값을 다시 저장하면 바뀐다.

```python
score = 80
print(score)   # 80

score = 95
print(score)   # 95
```

같은 이름으로 다시 저장하면 이전 값은 사라진다.

### 현재 값을 이용해서 바꾸기

```python
count = 0
print(count)   # 0

count = count + 1   # count의 현재 값(0)에 1을 더해서 다시 저장
print(count)   # 1

count = count + 1
print(count)   # 2
```

이 패턴은 매우 자주 쓰인다. 줄여서 쓸 수도 있다.

```python
count = 0
count += 1   # count = count + 1 과 같음
count += 1
print(count)  # 2

total = 100
total -= 20   # total = total - 20
print(total)  # 80

price = 50
price *= 2    # price = price * 2
print(price)  # 100
```

---

## 5. 여러 변수 한 번에 할당

```python
# 같은 값을 여러 변수에 동시에
a = b = c = 0
print(a, b, c)   # 0 0 0

# 여러 값을 여러 변수에 동시에
x, y, z = 1, 2, 3
print(x)   # 1
print(y)   # 2
print(z)   # 3
```

두 번째 방식은 특히 함수가 여러 값을 돌려줄 때 유용하다.

```python
# 이름과 나이를 동시에 할당
name, age = "Mina", 14
print(name)  # Mina
print(age)   # 14
```

---

## 6. 자주 쓰는 변수 패턴

### 카운터 (counter)

어떤 일이 몇 번 일어났는지 세는 변수다.

```python
count = 0           # 카운터 초기화

count += 1          # 한 번 발생
count += 1          # 또 한 번 발생
count += 1          # 또 한 번

print("총 횟수:", count)   # 총 횟수: 3
```

### 누산기 (accumulator)

값을 계속 더해나가는 변수다.

```python
total = 0           # 누산기 초기화

total += 1000       # 1000원 추가
total += 2500       # 2500원 추가
total += 500        # 500원 추가

print("합계:", total)   # 합계: 4000
```

### 임시 저장 (swap)

두 변수의 값을 서로 맞바꿀 때 사용한다.

```python
a = 10
b = 20

# 방법 1: 임시 변수 사용
temp = a    # a 값을 임시로 저장
a = b       # b 값을 a에 저장
b = temp    # 임시 저장된 값을 b에 저장

print(a)    # 20
print(b)    # 10

# 방법 2: Python 전용 한 줄 교환
a, b = b, a
```

---

## 7. 변수 타입 확인: type()

Python의 변수는 어떤 종류의 값을 담느냐에 따라 타입이 다르다. `type()` 함수로 확인할 수 있다.

```python
name = "Mina"
age = 14
height = 162.5
is_student = True

print(type(name))       # <class 'str'>
print(type(age))        # <class 'int'>
print(type(height))     # <class 'float'>
print(type(is_student)) # <class 'bool'>
```

| 타입 이름 | 의미 | 예시 |
|---------|------|------|
| `str` | 문자열 (텍스트) | `"Mina"`, `"hello"` |
| `int` | 정수 | `14`, `0`, `-5` |
| `float` | 소수 | `162.5`, `3.14` |
| `bool` | 참/거짓 | `True`, `False` |

타입이 다르면 할 수 있는 연산도 다르다.

```python
a = 10
b = 3
print(a + b)   # 13 (숫자 덧셈)

x = "Python"
y = " 공부"
print(x + y)   # Python 공부 (문자열 이어 붙이기)

# 숫자와 문자열은 더할 수 없다
print(a + x)   # TypeError!
```

타입 오류(TypeError)는 숫자와 문자열을 섞어서 연산할 때 자주 나온다. `type()`으로 확인하는 습관을 들이면 이런 오류를 빨리 찾을 수 있다.

---

## 8. 자주 겪는 실수

### NameError: 변수를 만들기 전에 사용

```python
print(score)   # NameError: name 'score' is not defined
score = 80
```

**해결:** 변수를 사용하기 전에 반드시 먼저 만들어야 한다.

```python
score = 80
print(score)   # 정상 작동
```

### 대소문자 혼동

```python
name = "Mina"
print(Name)    # NameError: name 'Name' is not defined
```

`name`과 `Name`은 Python에서 완전히 다른 변수다. 만들 때 쓴 이름과 똑같이 사용해야 한다.

### 따옴표 안에 변수명 쓰기

```python
name = "Mina"
print("name")   # name 이라는 글자가 그대로 출력됨 (변수 값이 아님)
print(name)     # Mina 출력 (변수 값)
```

### 변수 이름에 공백 또는 특수문자

```python
student name = "Kim"   # SyntaxError: 공백 불가
my-score = 90          # SyntaxError: 하이픈 불가

student_name = "Kim"   # 올바름
my_score = 90          # 올바름
```

### 초기화 없이 누산기 사용

```python
total += 1000   # NameError: total이 없음
```

**해결:** 누산기 변수는 반드시 0으로 초기화해야 한다.

```python
total = 0       # 먼저 초기화
total += 1000   # 정상 작동
```

---

## 9. 자주 하는 실수 정리표

| 실수 | 오류 메시지 예시 | 해결 방법 |
|------|----------------|----------|
| 변수를 만들기 전에 사용 | `NameError: name 'score' is not defined` | 사용하기 전에 먼저 `score = 값`으로 만들기 |
| `print("name")` — 따옴표 안에 변수명 | `name`이라는 글자가 그대로 출력됨 | 따옴표 없이 `print(name)`으로 쓰기 |
| 변수 이름에 공백 사용 | `SyntaxError` | 공백 대신 밑줄 사용 (`student_name`) |
| 대소문자 혼동 (`Name` vs `name`) | `NameError` | 만들 때 쓴 이름과 똑같이 사용 |
| 숫자와 문자열 더하기 | `TypeError` | `type()`으로 타입 확인 후 변환 |
| 예약어를 변수 이름으로 사용 | `SyntaxError` | 이름에 설명 단어 추가 (`if_value`) |
| 누산기 초기화 없이 사용 | `NameError` | `total = 0` 먼저 초기화 |

---

## 실습

### 실습 1: 학생 정보 저장하기 (따라 하기)

`student.py` 파일을 만들고 아래 코드를 입력한다.

```python
# 학생 정보를 변수에 저장한다
student_name = "김민준"
student_age = 15
student_grade = 2
student_score = 87.5
is_class_leader = False

# 저장된 정보를 출력한다
print("=== 학생 정보 ===")
print(f"이름: {student_name}")
print(f"나이: {student_age}세")
print(f"학년: {student_grade}학년")
print(f"평균 점수: {student_score}점")
print(f"반장 여부: {is_class_leader}")

# 타입도 확인해보자
print("\n=== 타입 확인 ===")
print(f"이름 타입: {type(student_name)}")
print(f"나이 타입: {type(student_age)}")
print(f"점수 타입: {type(student_score)}")
print(f"반장 여부 타입: {type(is_class_leader)}")
```

저장하고 실행한다.

```bash
python3 student.py
```

예상 출력:
```
=== 학생 정보 ===
이름: 김민준
나이: 15세
학년: 2학년
평균 점수: 87.5점
반장 여부: False

=== 타입 확인 ===
이름 타입: <class 'str'>
나이 타입: <class 'int'>
점수 타입: <class 'float'>
반장 여부 타입: <class 'bool'>
```

**직접 해보기:** 변수에 들어있는 값을 자신의 정보로 바꿔서 실행해보자. 이름, 나이, 학년, 점수를 모두 바꿔도 출력 형식은 그대로 유지된다는 것을 확인한다.

---

### 실습 2: 계산기 만들기 (따라 하기)

`calculator.py` 파일을 새로 만들고 아래를 입력한다.

```python
# 두 수를 변수에 저장한다
num1 = 15
num2 = 4

# 각 계산 결과를 변수에 저장한다
result_add = num1 + num2
result_sub = num1 - num2
result_mul = num1 * num2
result_div = num1 / num2        # 나눗셈 (소수 결과)
result_div_int = num1 // num2   # 몫 (정수 결과)
result_mod = num1 % num2        # 나머지

# 결과를 출력한다
print(f"{num1} + {num2} = {result_add}")
print(f"{num1} - {num2} = {result_sub}")
print(f"{num1} * {num2} = {result_mul}")
print(f"{num1} / {num2} = {result_div}")
print(f"{num1} // {num2} = {result_div_int} (몫)")
print(f"{num1} % {num2} = {result_mod} (나머지)")
```

예상 출력:
```
15 + 4 = 19
15 - 4 = 11
15 * 4 = 60
15 / 4 = 3.75
15 // 4 = 3 (몫)
15 % 4 = 3 (나머지)
```

**직접 해보기:** `num1`과 `num2` 값을 바꿔보자. 나눗셈에서 `num2 = 0`으로 바꾸면 어떤 오류가 나는지 확인해보자. 오류 메시지를 읽고 무슨 뜻인지 이해해보자.

아래처럼 누산기 패턴도 추가해보자.

```python
# 누산기 패턴: 여러 점수의 합계 구하기
total = 0
total += 85
total += 92
total += 78
total += 90

count = 4
average = total / count

print(f"\n총점: {total}")
print(f"평균: {average}")
```

---

### 실습 3: 변수 값 교환하기 (따라 하기)

`swap.py` 파일을 새로 만들고 아래를 입력한다.

```python
# 두 학생의 점수
student_a = 75
student_b = 90

print("교환 전:")
print(f"  학생 A: {student_a}점")
print(f"  학생 B: {student_b}점")

# 방법 1: 임시 변수를 이용한 교환
temp = student_a
student_a = student_b
student_b = temp

print("\n교환 후 (임시 변수 방법):")
print(f"  학생 A: {student_a}점")
print(f"  학생 B: {student_b}점")

# 다시 원래대로 되돌리기
student_a = 75
student_b = 90

# 방법 2: Python 전용 한 줄 교환
student_a, student_b = student_b, student_a

print("\n교환 후 (Python 방식):")
print(f"  학생 A: {student_a}점")
print(f"  학생 B: {student_b}점")
```

예상 출력:
```
교환 전:
  학생 A: 75점
  학생 B: 90점

교환 후 (임시 변수 방법):
  학생 A: 90점
  학생 B: 75점

교환 후 (Python 방식):
  학생 A: 90점
  학생 B: 75점
```

**직접 해보기:** 임시 변수 없이 교환하면 어떻게 될지 생각해보고 직접 실험해보자.

```python
a = 10
b = 20

# 임시 변수 없이 교환 시도 (잘못된 방법)
a = b   # a가 20이 됨
b = a   # b도 20이 됨 (원래 a 값인 10이 사라짐!)

print(a)  # 20
print(b)  # 20 (10이 아님!)
```

왜 `temp` 변수가 필요한지 체감할 수 있다.

---

## 확인 체크리스트

- [ ] `name = "Mina"` 처럼 변수를 만들 수 있는가
- [ ] `print(name)` 으로 변수 값을 출력할 수 있는가
- [ ] 변수에 새 값을 저장해서 바꿀 수 있는가
- [ ] 변수 이름의 규칙 (숫자 시작 금지, 공백 금지, snake_case)을 말할 수 있는가
- [ ] `print("name")` 과 `print(name)` 의 차이를 설명할 수 있는가
- [ ] `type()` 으로 변수의 타입을 확인할 수 있는가
- [ ] 카운터, 누산기 패턴을 직접 작성할 수 있는가
- [ ] 두 변수의 값을 교환하는 코드를 작성할 수 있는가
- [ ] NameError가 나면 원인을 찾을 수 있는가

---

## 한 번 더 생각해 보기

1. 변수를 쓰지 않고 같은 이름을 10군데 출력했다면, 이름이 바뀔 때 어떻게 해야 할까?
2. `score = 80` 다음에 `score = 90`을 쓰면, `score`의 값은 무엇인가?
3. `Score`와 `score`는 같은 변수일까 다른 변수일까?
4. `temp = a` → `a = b` → `b = temp` 순서에서 `temp`가 없으면 어떤 문제가 생기나?
5. `type(3.0)`과 `type(3)`은 같은 결과를 줄까?

---

## 다음 장

다음 장에서는 **자료형**을 배운다. Python에서 다루는 데이터에는 숫자, 글자, 참/거짓 등 여러 종류가 있다. 자료형을 알면 어떤 연산이 가능한지, 왜 `"3" + 3`이 오류를 내는지 이해할 수 있다.

---

## 참고 자료

- Python Tutorial: Introduction — https://docs.python.org/3/tutorial/introduction.html
- Python Built-in Types — https://docs.python.org/3/library/stdtypes.html
- Python Keywords — https://docs.python.org/3/reference/lexical_analysis.html#keywords
