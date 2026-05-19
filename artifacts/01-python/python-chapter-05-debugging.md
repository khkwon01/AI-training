# Chapter 05. 디버깅과 오류 메시지 읽기

## 이 장에서 배우는 것

- 오류가 왜 생기는지, 그리고 왜 무서워하지 않아도 되는지
- Traceback의 각 줄이 무슨 뜻인지 정확히 읽는 법
- SyntaxError, NameError, TypeError, IndexError, AttributeError 5가지 오류를 실제 코드와 함께
- `print()` 디버깅 방법과 VS Code 디버거 사용법
- print 디버깅 vs 디버거 — 언제 무엇을 쓰는지

---

## 왜 디버깅이 필요한가

모든 프로그래머는 오류를 만든다. 초보자도, 10년 경력자도 마찬가지다.

차이는 오류를 "빠르게 읽고 고치는 능력"에 있다.

오류 메시지는 Python이 "여기가 이상해요, 이런 이유로요"라고 알려주는 친절한 힌트다.  
오류 메시지를 읽는 법만 익히면 대부분의 문제는 5분 안에 해결할 수 있다.

---

## 1. Traceback이란 무엇인가

오류가 나면 Python은 아래와 같은 내용을 출력한다. 이것을 **Traceback**이라고 한다.

```python
def greet(name):
    print("안녕하세요, " + name + "!")

def start():
    greet(42)

start()
```

```
Traceback (most recent call last):
  File "test.py", line 7, in <module>
    start()
  File "test.py", line 5, in start
    greet(42)
  File "test.py", line 2, in greet
    print("안녕하세요, " + name + "!")
TypeError: can only concatenate str (not "int") to str
```

**Traceback 각 줄의 의미**:

| 줄 | 의미 |
|----|------|
| `Traceback (most recent call last):` | "호출 기록을 보여준다. 가장 최근 호출이 마지막에 있다" |
| `File "test.py", line 7, in <module>` | `test.py` 파일 7번째 줄, 최상위 코드에서 |
| `start()` | 그 줄의 코드 |
| `File "test.py", line 5, in start` | `test.py` 5번째 줄, `start` 함수 안에서 |
| `greet(42)` | 그 줄의 코드 |
| `File "test.py", line 2, in greet` | `test.py` 2번째 줄, `greet` 함수 안에서 |
| `print("안녕하세요, " + name + "!")` | 그 줄의 코드 |
| `TypeError: can only concatenate str (not "int") to str` | 오류 종류와 설명 |

**Traceback 읽는 순서**:
1. 맨 마지막 줄 — 오류 종류와 설명을 먼저 읽는다
2. 그 위 줄 — 오류가 실제로 발생한 파일과 줄 번호
3. 나머지 위로 올라가며 — 어떤 함수를 통해 거기에 도달했는지

대부분의 경우 **맨 아래 두 줄**만 읽어도 문제를 찾을 수 있다.

---

## 2. SyntaxError — 문법 오류

### 왜 생기는가

Python 코드를 실행하기 전에 Python이 코드를 파싱(해석)한다.  
괄호를 빠뜨리거나, 따옴표를 닫지 않거나, 들여쓰기가 틀리면 파싱 단계에서 실패한다.  
SyntaxError는 코드가 한 줄도 실행되기 전에 나온다.

### 실제 코드와 에러

**예시 1: 괄호 빠뜨리기**
```python
print("안녕하세요"
```
```
  File "test.py", line 1
    print("안녕하세요"
                     ^
SyntaxError: '(' was never closed
```

**예시 2: 따옴표 닫지 않기**
```python
name = "Mina
print(name)
```
```
  File "test.py", line 1
    name = "Mina
           ^
SyntaxError: EOL while scanning string literal
```

**예시 3: 들여쓰기 오류**
```python
def greet():
print("안녕")  # 들여쓰기가 없다
```
```
  File "test.py", line 2
    print("안녕")
    ^
IndentationError: expected an indented block after function definition
```

### 해결 방법

- 괄호, 따옴표가 열린 만큼 닫혔는지 확인한다
- `def`, `if`, `for`, `while` 뒤에는 반드시 들여쓴 블록이 있어야 한다
- VS Code에서는 문법 오류를 빨간 밑줄로 미리 표시해준다

---

## 3. NameError — 이름을 찾지 못함

### 왜 생기는가

Python이 변수나 함수 이름을 찾지 못할 때 발생한다.  
변수를 만들기 전에 사용하거나, 이름을 다르게 적었을 때 생긴다.

### 실제 코드와 에러

```python
print(user_name)
```
```
NameError: name 'user_name' is not defined
```

**대소문자 오타**:
```python
student_name = "Mina"
print(Student_name)  # 대문자 S
```
```
NameError: name 'Student_name' is not defined
```

**함수를 정의하기 전에 사용**:
```python
result = calculate(10, 20)

def calculate(a, b):
    return a + b
```
```
NameError: name 'calculate' is not defined
```

### 해결 방법

- 변수 이름의 대소문자를 정확히 확인한다
- 변수나 함수를 사용하기 전에 먼저 정의했는지 확인한다
- `print(dir())` 로 현재 범위에서 사용 가능한 이름 목록을 볼 수 있다

---

## 4. TypeError — 자료형 오류

### 왜 생기는가

잘못된 자료형으로 연산하거나 함수를 호출할 때 발생한다.

### 실제 코드와 에러

**문자열과 숫자를 더하기**:
```python
age = 14
message = "나이는 " + age
```
```
TypeError: can only concatenate str (not "int") to str
```

**해결**:
```python
age = 14
message = "나이는 " + str(age)
# 또는
message = f"나이는 {age}"
```

**함수 인수 개수 오류**:
```python
def add(a, b):
    return a + b

result = add(1, 2, 3)  # 인수가 3개인데 함수는 2개를 받는다
```
```
TypeError: add() takes 2 positional arguments but 3 were given
```

**None에 연산하기**:
```python
def get_name():
    name = "Mina"
    # return을 빠뜨렸다

result = get_name()
print(result.upper())  # result는 None
```
```
AttributeError: 'NoneType' object has no attribute 'upper'
```

(이 경우는 TypeError가 아닌 AttributeError가 나지만, return을 빠뜨리는 것이 원인이다)

### 해결 방법

- 문자열과 숫자를 합칠 때는 `str()`로 변환하거나 f-string을 사용한다
- 함수가 받는 인수 개수와 실제로 넣는 개수가 맞는지 확인한다

---

## 5. IndexError — 범위를 벗어난 인덱스

### 왜 생기는가

리스트나 문자열에서 존재하지 않는 위치(인덱스)를 읽으려 할 때 발생한다.

### 실제 코드와 에러

```python
numbers = [10, 20, 30]
print(numbers[5])
```
```
IndexError: list index out of range
```

**리스트는 0번부터 시작한다는 점을 잊는 경우**:
```python
fruits = ["사과", "바나나", "딸기"]
print(fruits[3])  # 0, 1, 2까지만 있다. 3은 없다
```
```
IndexError: list index out of range
```

**빈 리스트를 읽으려는 경우**:
```python
scores = []
print(scores[0])  # 빈 리스트에는 아무것도 없다
```
```
IndexError: list index out of range
```

### 해결 방법

```python
numbers = [10, 20, 30]

# 방법 1: 길이 확인
print(len(numbers))  # 3 → 인덱스는 0, 1, 2까지만 유효

# 방법 2: 조건으로 체크
index = 5
if index < len(numbers):
    print(numbers[index])
else:
    print("해당 인덱스가 없습니다")

# 방법 3: try/except
try:
    print(numbers[5])
except IndexError:
    print("인덱스가 범위를 벗어났습니다")
```

---

## 6. AttributeError — 속성이나 메서드가 없음

### 왜 생기는가

객체에 없는 속성이나 메서드를 호출할 때 발생한다.  
자료형을 착각하거나, None을 반환하는 함수의 결과에 메서드를 쓸 때 자주 생긴다.

### 실제 코드와 에러

**숫자에 문자열 메서드 사용**:
```python
number = 123
print(number.upper())  # 숫자에는 upper() 메서드가 없다
```
```
AttributeError: 'int' object has no attribute 'upper'
```

**None에 메서드 사용 (return을 빠뜨린 경우)**:
```python
def get_greeting(name):
    greeting = f"안녕하세요, {name}!"
    # return을 빠뜨렸다

result = get_greeting("Mina")
print(result.upper())  # result가 None이다
```
```
AttributeError: 'NoneType' object has no attribute 'upper'
```

**리스트를 딕셔너리로 착각**:
```python
data = ["Mina", 14, "Seoul"]
print(data["name"])  # 리스트는 숫자 인덱스로만 접근한다
```
```
TypeError: list indices must be integers or slices, not str
```

### 해결 방법

- `type(변수)` 로 자료형을 먼저 확인한다
- 함수에서 `return`을 빠뜨리지 않았는지 확인한다
- `print(dir(변수))` 로 해당 객체에서 사용 가능한 메서드 목록을 볼 수 있다

```python
name = "Mina"
print(type(name))    # <class 'str'>
print(dir(name))     # 사용 가능한 메서드 목록 출력
```

---

## 7. print() 디버깅 — 가장 빠른 방법

### 왜 print()로 디버깅하는가

코드가 예상대로 동작하지 않을 때, 중간중간 값을 출력해서 어디서 어긋나는지 확인하는 방법이다.  
설치나 설정이 필요 없고, 누구나 바로 쓸 수 있다.

### 기본 사용법

```python
def calculate_average(scores):
    print(f"입력된 점수: {scores}")      # 입력값 확인
    total = sum(scores)
    print(f"합계: {total}")             # 중간 계산 확인
    count = len(scores)
    print(f"개수: {count}")             # 중간 계산 확인
    average = total / count
    print(f"평균: {average}")           # 결과 확인
    return average

result = calculate_average([80, 90, 70])
print(f"최종 결과: {result}")
```

```
입력된 점수: [80, 90, 70]
합계: 240
개수: 3
평균: 80.0
최종 결과: 80.0
```

### 자료형도 함께 출력하기

```python
value = "14"
print(f"value = {value}, 타입 = {type(value)}")
# value = 14, 타입 = <class 'str'>
# 숫자처럼 보이지만 실제로는 문자열이다
```

### print 디버깅의 한계

- 중간에 `print`를 많이 넣으면 코드가 지저분해진다
- 디버깅이 끝나면 `print`를 하나씩 지워야 한다
- 복잡한 로직에서는 어디에 `print`를 넣어야 할지 판단하기 어렵다

---

## 8. VS Code 디버거 사용법

### 왜 디버거를 쓰는가

print 디버깅은 간단하지만, 복잡한 코드에서는 한계가 있다.  
VS Code 디버거를 사용하면 코드를 한 줄씩 실행하면서 모든 변수의 값을 실시간으로 볼 수 있다.

### 중단점(Breakpoint) 설정하기

1. VS Code에서 Python 파일을 열기
2. 멈추고 싶은 줄 번호 왼쪽을 클릭하면 빨간 점(중단점)이 생긴다
3. `F5` 또는 상단 메뉴 Run → Start Debugging 클릭

중단점에 도달하면 코드 실행이 일시 정지되고, 왼쪽 패널에 현재 모든 변수의 값이 표시된다.

### 주요 디버거 단축키

| 단축키 | 기능 | 설명 |
|--------|------|------|
| `F5` | Continue | 다음 중단점까지 계속 실행 |
| `F10` | Step Over | 현재 줄을 실행하고 다음 줄로 이동 (함수 안으로 들어가지 않음) |
| `F11` | Step Into | 현재 줄을 실행하되, 함수를 호출하면 그 함수 안으로 들어감 |
| `Shift+F11` | Step Out | 현재 함수를 끝까지 실행하고 호출한 곳으로 돌아감 |
| `Shift+F5` | Stop | 디버깅 종료 |

### Step Over vs Step Into

```python
def add(a, b):
    return a + b          # Step Into를 하면 이 줄로 들어온다

result = add(10, 20)      # 이 줄에서 F10(Step Over)을 하면 add()를 건너뛰고 다음 줄로
                          # F11(Step Into)을 하면 add() 함수 안으로 들어간다
print(result)
```

- **Step Over(F10)**: 함수를 블랙박스처럼 처리. 함수 내부는 보지 않고 결과만 얻고 넘어간다.
- **Step Into(F11)**: 함수 내부로 들어가서 한 줄씩 실행한다. 함수 안이 의심스러울 때 사용한다.

---

## 9. print 디버깅 vs 디버거 — 언제 무엇을 쓰는가

| 상황 | 추천 방법 |
|------|-----------|
| 빠르게 값 하나만 확인하고 싶다 | print 디버깅 |
| 코드가 짧고 단순하다 | print 디버깅 |
| 복잡한 로직에서 흐름을 따라가고 싶다 | 디버거 |
| 어떤 함수에서 문제가 생기는지 모른다 | 디버거 |
| 반복문 안에서 특정 조건일 때 멈추고 싶다 | 디버거 (조건부 중단점) |
| 다른 사람의 코드를 이해하려 한다 | 디버거 |

초보자는 print 디버깅부터 시작하는 것이 좋다.  
어느 정도 익숙해지면 디버거를 써보는 것을 권장한다.

---

## 실습 1 (따라 하기). NameError 고치기

**오류가 있는 코드**:
```python
print(student_name)
```

**직접 실행하면 나오는 에러**:
```
NameError: name 'student_name' is not defined
```

**따라 하기**:
1. 에러 메시지를 읽는다: `student_name`이라는 이름이 정의되지 않았다
2. `student_name` 변수를 먼저 만든다

**수정된 코드**:
```python
student_name = "Mina"
print(student_name)
```

**직접 해보기**: 변수 이름을 `student_name`으로 만들고 `print(studentname)` (언더바 없이)를 출력해보자. 어떤 에러가 나는가? 이유는 무엇인가?

---

## 실습 2 (따라 하기). TypeError 고치기

**오류가 있는 코드**:
```python
age = 15
score = 92
print("이름: Jisoo, 나이: " + age + ", 점수: " + score)
```

**직접 실행하면 나오는 에러**:
```
TypeError: can only concatenate str (not "int") to str
```

**따라 하기**:
1. 에러 메시지를 읽는다: 문자열(`str`)에 정수(`int`)를 이어붙일 수 없다
2. 두 가지 해결 방법 중 하나를 고른다

**방법 1: str() 변환**:
```python
age = 15
score = 92
print("이름: Jisoo, 나이: " + str(age) + ", 점수: " + str(score))
```

**방법 2: f-string (더 깔끔하다)**:
```python
age = 15
score = 92
print(f"이름: Jisoo, 나이: {age}, 점수: {score}")
```

**직접 해보기**: `age = "15"` (따옴표 있음)으로 바꾸면 에러가 사라지는가? 그렇다면 왜인가?

---

## 실습 3 (따라 하기). print()로 중간값 확인하기

**동작하지만 결과가 이상한 코드**:
```python
def calculate_discount(price, discount_rate):
    discounted = price - discount_rate
    return discounted

original_price = 10000
discount = 0.1  # 10% 할인
final_price = calculate_discount(original_price, discount)
print(f"할인 후 가격: {final_price}")
# 기대값: 9000, 실제값: 9999.9
```

**따라 하기 — print로 중간값 확인하기**:
```python
def calculate_discount(price, discount_rate):
    print(f"  입력 price: {price}, 타입: {type(price)}")
    print(f"  입력 discount_rate: {discount_rate}, 타입: {type(discount_rate)}")
    discounted = price - discount_rate
    print(f"  계산 결과: {discounted}")
    return discounted

original_price = 10000
discount = 0.1
final_price = calculate_discount(original_price, discount)
print(f"할인 후 가격: {final_price}")
```

```
  입력 price: 10000, 타입: <class 'int'>
  입력 discount_rate: 0.1, 타입: <class 'float'>
  계산 결과: 9999.9
할인 후 가격: 9999.9
```

print를 보면 `0.1`이 10%가 아니라 0.1원으로 계산되고 있다는 것을 알 수 있다.

**수정된 코드**:
```python
def calculate_discount(price, discount_rate):
    discounted = price * (1 - discount_rate)  # 0.1 → 10% 할인
    return discounted

original_price = 10000
discount = 0.1
final_price = calculate_discount(original_price, discount)
print(f"할인 후 가격: {final_price}")
# 할인 후 가격: 9000.0
```

**직접 해보기**: `discount = 0.2` (20% 할인)로 바꾸면 결과가 얼마가 되어야 하는가? 직접 실행해서 확인해보자.

---

## 자주 막히는 지점 (Common Pitfalls)

### Pitfall 1. 에러 메시지를 읽지 않고 바로 포기한다

에러 메시지는 문제를 해결하는 힌트다. 영어라도 마지막 줄만 읽으면 대부분 이해된다.  
`NameError`, `TypeError`, `IndexError` — 이 단어만 알아도 90%는 해결된다.

---

### Pitfall 2. 아무 줄이나 고친다

Traceback에서 파일 이름과 줄 번호를 확인하고, 정확히 그 줄부터 살펴본다.

---

### Pitfall 3. 한 번에 여러 군데를 고친다

한 번에 한 곳만 고치고 실행해보는 것이 원칙이다.  
여러 곳을 동시에 바꾸면 어떤 변경이 문제를 고쳤는지 알 수 없다.

---

### Pitfall 4. return을 빠뜨린다

```python
def get_greeting(name):
    greeting = f"안녕하세요, {name}!"
    # return이 없으면 함수는 None을 반환한다

result = get_greeting("Mina")
print(result)  # None 출력
print(result.upper())  # AttributeError 발생
```

함수가 예상한 값을 반환하지 않으면 `return`을 빠뜨린 것이 아닌지 확인한다.

---

### Pitfall 5. IndentationError를 문법 오류가 아니라고 생각한다

IndentationError는 SyntaxError의 한 종류다. 탭과 스페이스를 섞어 쓰면 발생하기 쉽다.  
VS Code에서는 항상 스페이스 4칸(또는 탭)으로 통일하는 것이 좋다.

---

## 확인 체크리스트

- Traceback에서 오류 종류, 파일 이름, 줄 번호를 찾을 수 있는가
- SyntaxError, NameError, TypeError, IndexError, AttributeError 각각의 원인을 말할 수 있는가
- `print()`로 중간값을 확인하는 방법을 사용할 수 있는가
- VS Code에서 중단점을 설정하고 Step Over/Into를 사용할 수 있는가
- print 디버깅과 디버거 중 언제 무엇을 쓰는지 설명할 수 있는가

---

## 한 번 더 생각해 보기

1. `TypeError: can only concatenate str (not "int") to str` 에서 왼쪽이 str이 아닌 int면 어떤 에러가 날까? 직접 실험해보자.
2. `print(type(변수))` 는 언제 가장 유용하게 쓰일까?
3. 디버거의 "Step Over"와 "Step Into"는 어떤 상황에서 각각 선택하겠는가?
4. 오류 메시지를 처음 봤을 때 읽는 순서 3단계를 자신의 말로 정리해보자.

---

## 교사용 메모

- 강조: 오류는 실패가 아니라 힌트다. Traceback 맨 아래 줄부터 읽는 습관을 만들어준다.
- 막힘 포인트: 문제 줄이 아닌 다른 줄을 고치는 경우, TypeError를 자료형 문제로 연결하지 못하는 경우, `return` 누락으로 생기는 AttributeError를 이해하지 못하는 경우가 많다.
- 실습 3에서 "기대값과 실제값이 다른" 상황을 먼저 경험하게 하고, 그 다음에 print 디버깅으로 원인을 찾는 과정을 보여주면 효과적이다.
- 질문 1: Traceback을 받으면 가장 먼저 어느 줄을 읽는가?
- 질문 2: `return`을 빠뜨렸을 때 함수는 무엇을 반환하는가?
