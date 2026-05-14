# ai-train Python Basics Chapter 02: 변수

## 이 장에서 배우는 것

- 변수가 무엇인지, 왜 필요한지
- 변수를 만들고 값을 저장하는 방법
- 변수에 저장된 값을 출력하는 방법
- 변수 이름을 짓는 규칙

---

## 먼저 쉬운 설명

이전 장에서는 이렇게 썼다.

```python
print("이름: Mina")
print("나이: 14")
```

만약 이름을 10군데서 출력해야 한다면? 매번 `"Mina"`를 타이핑해야 한다. 나중에 이름이 바뀌면 10군데를 전부 고쳐야 한다.

변수는 이 문제를 해결한다. 값을 이름 붙여 저장해 두면, 나중에 그 이름으로 꺼내 쓸 수 있다.

```python
name = "Mina"
print(name)
print(name)
print(name)
```

이름을 바꾸고 싶을 때는 한 줄만 수정하면 된다.

---

## 1. 변수 만들기

변수는 `이름 = 값` 형태로 만든다.

```python
name = "Mina"
age = 14
```

- `name`은 변수 이름이다
- `=`는 오른쪽 값을 왼쪽 이름에 저장하라는 의미다
- `"Mina"`는 저장하는 값이다

수학의 `=` (같다)와 다르다. Python의 `=`는 "저장하라"는 뜻이다.

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
print(name)    # Mina 출력
print("name")  # name 이라는 글자 출력
```

---

## 3. 변수와 글자를 함께 출력하기

`print()` 안에 여러 값을 콤마로 구분하면 함께 출력된다.

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

---

## 4. 변수 값 바꾸기

변수는 값을 다시 저장하면 바뀐다.

```python
score = 80
print(score)

score = 95
print(score)
```

출력:

```
80
95
```

같은 이름으로 다시 저장하면 이전 값은 사라진다.

---

## 5. 변수 이름 짓는 규칙

변수 이름은 아무렇게나 짓지 않는다. 규칙이 있다.

**규칙**:
- 영문자, 숫자, 밑줄(`_`)만 사용한다
- 숫자로 시작하면 안 된다
- 공백을 쓸 수 없다 (공백 대신 `_` 사용)
- 대소문자를 구분한다 (`name`과 `Name`은 다른 변수)

좋은 이름 예:
```python
student_name = "Jisoo"
total_score = 100
item_count = 5
```

나쁜 이름 예:
```python
1score = 90       # 숫자로 시작 → 오류
student name = "Kim"  # 공백 → 오류
s = "Kim"         # 너무 짧아서 의미를 알 수 없음
```

이름은 짧아도 안 되고 너무 길어도 불편하다. **무엇을 담는 변수인지 바로 알 수 있는 이름**이 가장 좋다.

---

## 6. 따라 하기 실습

앞 장에서 만든 `intro.py` 파일을 변수를 사용하는 방식으로 고쳐 보자.

### 실습 1. 변수 사용해서 자기소개 출력하기

`intro.py` 파일을 열고 아래처럼 수정한다.

```python
name = "Mina"
age = 14
favorite = "Python"

print("이름:", name)
print("나이:", age)
print("배우고 싶은 것:", favorite)
```

저장하고 실행한다.

```bash
python3 intro.py
```

예상 출력:

```
이름: Mina
나이: 14
배우고 싶은 것: Python
```

### 실습 2. 값을 바꿔서 다시 출력하기

`intro.py` 아래에 이어서 써보자. 나이를 바꾸고 다시 출력한다.

```python
age = 15
print("업데이트된 나이:", age)
```

저장하고 실행해서 결과를 확인한다.

### 실습 3. 두 변수를 더하기

숫자 변수 두 개를 만들어서 합계를 출력해 보자.

`calc.py` 파일을 새로 만들고 아래를 입력한다.

```python
score1 = 80
score2 = 90
total = score1 + score2
print("합계:", total)
```

---

## 자주 하는 실수

| 실수 | 오류 메시지 예시 | 해결 방법 |
|------|----------------|----------|
| 변수를 만들기 전에 사용 | `NameError: name 'score' is not defined` | 사용하기 전에 먼저 `score = 값` 으로 만들기 |
| `print("name")` — 따옴표 안에 변수명 | `name` 이라는 글자가 그대로 출력됨 | 따옴표 없이 `print(name)` 으로 쓰기 |
| 변수 이름에 공백 사용 (`student name = ...`) | `SyntaxError` | 공백 대신 밑줄 사용 (`student_name`) |
| 대소문자 혼동 (`Name` vs `name`) | `NameError` | 만들 때 쓴 이름과 똑같이 사용 |

---

## 확인 체크리스트

- [ ] `name = "Mina"` 처럼 변수를 만들 수 있는가
- [ ] `print(name)` 으로 변수 값을 출력할 수 있는가
- [ ] 변수에 새 값을 저장해서 바꿀 수 있는가
- [ ] 변수 이름의 규칙(숫자 시작 금지, 공백 금지)을 말할 수 있는가
- [ ] `print("name")` 과 `print(name)` 의 차이를 설명할 수 있는가

---

## 한 번 더 생각해 보기

1. 변수를 쓰지 않고 같은 이름을 10군데 출력했다면, 이름이 바뀔 때 어떻게 해야 할까?
2. `score = 80` 다음에 `score = 90` 을 쓰면, `score`의 값은 무엇인가?
3. `Score`와 `score`는 같은 변수일까 다른 변수일까?

---

## 다음 장

다음 장에서는 **자료형**을 배운다. Python에서 다루는 데이터에는 숫자, 글자, 참/거짓 등 여러 종류가 있다. 자료형을 알면 어떤 연산이 가능한지 이해할 수 있다.

---

## 참고 자료

- Python Tutorial: Introduction — https://docs.python.org/3/tutorial/introduction.html
- Python Built-in Types — https://docs.python.org/3/library/stdtypes.html
