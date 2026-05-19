# Chapter 13: 함수

## 이 장에서 배우는 것

- 함수가 왜 필요한지
- `def`로 함수를 만들고 호출하는 방법
- 매개변수와 반환값 사용하기
- 기본값 매개변수와 여러 값 반환하기
- 함수를 쓸 때 주의할 점

---

## 먼저 쉬운 설명

같은 코드를 여러 곳에서 쓴다면 어떻게 할까?

```python
# 같은 코드가 여러 곳에 반복됨
print("안녕하세요, Mina님!")
print("안녕하세요, Jisoo님!")
print("안녕하세요, Tom님!")
```

함수로 묶으면:

```python
def greet(name):
    print(f"안녕하세요, {name}님!")

greet("Mina")
greet("Jisoo")
greet("Tom")
```

함수는 **이름 붙인 코드 블록**이다. 한 번 만들어두면 필요할 때 이름으로 불러서 실행할 수 있다.

---

## 1. 함수 만들기

```python
def 함수이름(매개변수):
    실행할 코드
```

예:

```python
def say_hello():
    print("안녕하세요!")

say_hello()   # 함수 호출
```

- `def` 로 시작
- 함수 이름 뒤에 `()` 와 `:` 필수
- 실행할 코드는 들여쓰기

---

## 2. 매개변수 (Parameter)

함수에 값을 전달하는 입력 창구다.

```python
def greet(name):        # name = 매개변수 (parameter): 함수를 정의할 때 쓰는 변수
    print(f"안녕하세요, {name}님!")

greet("Mina")           # "Mina" = 인수 (argument): 함수를 호출할 때 전달하는 실제 값
greet("Jisoo")          # "Jisoo" = 인수
```

- **매개변수(parameter)**: `def` 함수 정의할 때의 변수 이름 (`name`)
- **인수(argument)**: 함수를 호출할 때 전달하는 실제 값 (`"Mina"`)

두 용어를 혼용하는 경우가 많지만, 의미는 다르다.

여러 매개변수:

```python
def introduce(name, age):
    print(f"저는 {name}이고, {age}살입니다.")

introduce("Mina", 14)
```

---

## 3. 반환값 (Return)

함수가 계산한 결과를 돌려줄 때 `return`을 사용한다.

```python
def add(a, b):
    return a + b

result = add(3, 5)
print(result)   # 8
```

`return`이 실행되는 순간 함수는 즉시 종료된다. `return` 이후의 코드는 절대 실행되지 않는다.

```python
def add(a, b):
    return a + b
    print("이 줄은 절대 실행 안 됨")  # return 이후라서 무시
```

`return`이 없으면 함수는 `None`을 반환한다.

```python
def greet(name):
    print(f"안녕하세요, {name}님!")
    # return 없음

value = greet("Mina")
print(value)   # None
```

**print와 return의 차이:**

| | print | return |
|--|-------|--------|
| 역할 | 화면에 출력 | 값을 돌려줌 |
| 나중에 사용 가능 | 불가 | 가능 |
| 예시 | `print(result)` | `x = add(3, 5)` |

---

## 4. 기본값 매개변수

매개변수에 기본값을 설정하면 인수 없이도 호출할 수 있다.

```python
def greet(name, greeting="안녕하세요"):
    print(f"{greeting}, {name}님!")

greet("Mina")                    # 안녕하세요, Mina님!
greet("Tom", "Hello")            # Hello, Tom님!
```

기본값이 있는 매개변수는 기본값이 없는 것 뒤에 와야 한다.

---

## 5. 여러 값 반환

`return`으로 여러 값을 동시에 반환할 수 있다.

```python
def min_max(numbers):
    return min(numbers), max(numbers)

low, high = min_max([3, 1, 4, 1, 5, 9])
print(f"최솟값: {low}, 최댓값: {high}")
```

---

## 6. 앞 장 코드를 함수로 리팩토링

Chapter 11에서 만든 성적 등급 코드를 함수로 개선한다.

**이전 코드:**

```python
score = 72
if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
else:
    grade = "F"
```

**함수로 개선:**

```python
def get_grade(score):
    if score >= 90:
        return "A"
    elif score >= 80:
        return "B"
    elif score >= 70:
        return "C"
    else:
        return "F"

scores = [95, 72, 88, 61]
for score in scores:
    grade = get_grade(score)
    print(f"{score}점 → {grade}")
```

---

## 7. 따라 하기 실습

### 실습 1. 계산기 함수 만들기

`calculator.py` 파일을 만들고:

```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        return "0으로 나눌 수 없습니다"
    return a / b

print(add(10, 3))        # 13
print(subtract(10, 3))   # 7
print(multiply(10, 3))   # 30
print(divide(10, 3))     # 3.333...
print(divide(10, 0))     # 0으로 나눌 수 없습니다
```

### 실습 2. 앞 장 메모 서비스에 함수 추가

`memo.py`에 메모 개수를 반환하는 함수를 추가한다.

```python
def count_memos():
    return len(memos)

print(f"현재 메모 수: {count_memos()}")
```

### 실습 3. 함수가 있는 코드와 없는 코드 비교

같은 기능을 함수 없이 작성하고, 함수로 리팩토링해서 어떤 점이 좋아졌는지 직접 확인한다.

---

## 자주 하는 실수

| 실수 | 증상 | 해결 방법 |
|------|------|----------|
| 함수 만들고 호출 안 함 | 아무것도 실행 안 됨 | `def` 뒤에 `함수이름()` 으로 호출 |
| `print`와 `return` 혼용 | 반환값이 `None` | 결과를 다시 쓸 거면 `return`, 화면 출력만이면 `print` |
| 기본값 매개변수 순서 오류 | `SyntaxError` | 기본값 없는 매개변수를 앞에 배치 |
| 들여쓰기 오류 | `IndentationError` | `def:` 다음 코드는 4칸 들여쓰기 |

---

## 확인 체크리스트

- [ ] `def`로 함수를 만들고 호출할 수 있는가
- [ ] 매개변수와 반환값의 차이를 설명할 수 있는가
- [ ] `print`와 `return`의 차이를 말할 수 있는가
- [ ] 기존 코드를 함수로 리팩토링할 수 있는가

---

## 한 번 더 생각해 보기

1. 함수를 쓰지 않고 코드를 반복하면 어떤 문제가 생길까?
2. `return` 없는 함수가 반환하는 `None`은 어떻게 활용할 수 있을까?
3. 함수 하나에 너무 많은 기능을 넣으면 어떤 단점이 있을까?

---

## 다음 장

다음 장에서는 class와 object를 배운다. 함수로 동작을 묶었다면, class는 데이터와 동작을 함께 묶는 방법이다.

---

## 참고 자료

- Python Tutorial: Defining Functions — https://docs.python.org/3/tutorial/controlflow.html#defining-functions
