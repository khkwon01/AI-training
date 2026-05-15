# ai-train Python Chapter 11: 조건문

## 이 장에서 배우는 것

- 조건문이 무엇인지, 왜 필요한지
- `if`, `elif`, `else`로 상황에 따라 다른 코드를 실행하는 방법
- 비교 연산자와 논리 연산자 사용하기
- 조건문을 중첩하거나 연결하는 방법
- 초보자가 자주 하는 실수와 해결법

---

## 먼저 쉬운 설명

일상에서도 우리는 항상 조건에 따라 다르게 행동한다.

> "비가 오면 우산을 가져가고, 안 오면 그냥 나간다."

Python에서 이것을 표현하면:

```python
if 비가_온다:
    우산을_가져간다()
else:
    그냥_나간다()
```

조건문은 "이 조건이 참이면 이걸 해라, 아니면 저걸 해라"를 코드로 표현하는 방법이다.

---

## 1. if 기본 구조

```python
score = 85

if score >= 80:
    print("합격입니다")
```

- `if` 다음에 조건을 쓴다
- 조건 끝에 `:` 를 붙인다
- 실행할 코드는 **들여쓰기(4칸 또는 Tab)** 로 안쪽에 쓴다

조건이 `True`이면 들여쓰기된 코드가 실행된다. `False`이면 건너뛴다.

---

## 2. if / else

조건이 거짓일 때 실행할 코드를 `else`로 추가한다.

```python
score = 65

if score >= 80:
    print("합격입니다")
else:
    print("불합격입니다")
```

출력:
```
불합격입니다
```

---

## 3. if / elif / else

여러 조건을 순서대로 확인할 때는 `elif`를 사용한다.

```python
score = 72

if score >= 90:
    print("A")
elif score >= 80:
    print("B")
elif score >= 70:
    print("C")
else:
    print("F")
```

출력:
```
C
```

`elif`는 "else if"의 줄임이다. 위에서부터 순서대로 확인하다가 처음으로 `True`인 블록만 실행한다.

---

## 4. 비교 연산자

| 연산자 | 의미 | 예시 | 결과 |
|--------|------|------|------|
| `==` | 같다 | `5 == 5` | `True` |
| `!=` | 다르다 | `5 != 3` | `True` |
| `>` | 크다 | `5 > 3` | `True` |
| `<` | 작다 | `5 < 3` | `False` |
| `>=` | 크거나 같다 | `5 >= 5` | `True` |
| `<=` | 작거나 같다 | `3 <= 5` | `True` |

주의: `=`는 저장, `==`는 비교다.

```python
x = 10       # x에 10을 저장
x == 10      # x가 10과 같은지 확인 → True
```

---

## 5. 논리 연산자

여러 조건을 동시에 확인할 때 사용한다.

```python
age = 20
has_ticket = True

# and: 둘 다 True여야 True
if age >= 18 and has_ticket:
    print("입장 가능합니다")

# or: 하나라도 True이면 True
if age < 18 or not has_ticket:
    print("입장 불가합니다")

# not: True → False, False → True
if not has_ticket:
    print("티켓이 없습니다")
```

---

## 6. 문자열 조건

문자열도 조건에 사용할 수 있다.

```python
name = "Mina"

if name == "Mina":
    print("안녕하세요, Mina님!")

# in: 포함 여부 확인
language = "Python is fun"
if "Python" in language:
    print("Python 관련 문장입니다")
```

---

## 7. 따라 하기 실습

앞에서 만든 `memo.py` 대신 새 파일 `grade.py`를 만든다.

### 실습 1. 점수별 등급 출력

```python
score = int(input("점수를 입력하세요: "))

if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
elif score >= 60:
    grade = "D"
else:
    grade = "F"

print(f"점수: {score}, 등급: {grade}")
```

실행 후 여러 점수를 입력해서 결과를 확인한다.

### 실습 2. 로그인 시뮬레이션

```python
correct_password = "python123"
input_password = input("비밀번호를 입력하세요: ")

if input_password == correct_password:
    print("로그인 성공!")
else:
    print("비밀번호가 틀렸습니다.")
```

---

## 자주 하는 실수

| 실수 | 오류 / 증상 | 해결 방법 |
|------|------------|----------|
| `=` 와 `==` 혼동 | `SyntaxError` 또는 의도치 않은 동작 | 비교는 `==`, 저장은 `=` |
| 들여쓰기 빠짐 | `IndentationError` | `if:` 다음 줄은 반드시 4칸 들여쓰기 |
| `:` 빠짐 | `SyntaxError: expected ':'` | `if 조건:` 끝에 콜론 추가 |
| `elif` 순서 잘못됨 | 조건이 예상과 다르게 실행됨 | 더 좁은 조건을 위에 배치 |

**elif 순서 오류 예시:**

```python
score = 85

# 잘못된 순서 → 85점도 "B"가 아닌 "C 이상" 그룹으로 먼저 걸림
if score >= 80:
    print("B 이상")      # 85가 여기서 걸림
elif score >= 85:
    print("A- 이상")     # 이 조건은 절대 실행 안 됨

# 올바른 순서 → 더 좁은 조건(높은 점수)을 위에
if score >= 85:
    print("A- 이상")     # 85 → 여기서 걸림
elif score >= 80:
    print("B 이상")
```

좁은 범위(큰 숫자)를 항상 위에 배치한다.

---

## 확인 체크리스트

- [ ] `if`, `elif`, `else` 구조를 설명할 수 있는가
- [ ] `==` 과 `=` 의 차이를 말할 수 있는가
- [ ] `and`, `or`, `not` 을 언제 쓰는지 아는가
- [ ] 점수에 따라 다른 등급을 출력하는 코드를 직접 작성할 수 있는가

---

## 한 번 더 생각해 보기

1. `if score >= 80 and score < 90:` 을 더 짧게 쓰는 방법이 있을까?
2. `elif`가 없이 `if`를 여러 개 쓰면 어떻게 다를까?
3. 빈 문자열 `""` 은 조건에서 `True`일까 `False`일까?

---

## 다음 장

다음 장에서는 반복문을 배운다. `for`와 `while`로 같은 작업을 여러 번 반복하는 방법을 익힌다.

---

## 참고 자료

- Python Tutorial: More Control Flow Tools — https://docs.python.org/3/tutorial/controlflow.html
